<template>
  <hr class="border border-grey border-1 opacity-50" />
  <div class="container text-center">
    <div class="row">
      <div class="col-lg-5 mb-3 col-md-12 ">
        <img class="img-fluid" :src="dataCourse.course.thumbnailUrl" alt="" />
      </div>
      <div class="col-lg-7  col-md-12 text-start">
        <p>
          <span class="fw-bold">Tên khoá học:</span>
          {{ dataCourse.course.name }}
        </p>
        <p>
          <span class="fw-bold">Mô tả khoá học:</span>
          {{ dataCourse.course.description }}
        </p>
        <p>
          <span class="fw-bold">Giá khoá học:</span>
          {{
            formatCurrency(
              dataCourse.course.price,
              dataCourse.course.currencyUnit
            )
          }}
        </p>
        <p>
          <span class="fw-bold">Đơn vị:</span>
          {{ dataCourse.course.currencyUnit }}
        </p>
        <p>
          <span class="fw-bold">Công nghệ: </span>
          <span v-if="
            dataCourse.course &&
            dataCourse.course.techStack &&
            dataCourse.course.techStack.length > 0
          ">
            {{
              dataCourse.course.techStack.map((stack) => stack.name).join(", ")
            }}
          </span>
          <span v-else>N/A</span>
        </p>
        <p>
          <span class="fw-bold">Giảng viên: </span>
          <span v-if="
            dataCourse.course &&
            dataCourse.course.teacher &&
            dataCourse.course.teacher.length > 0
          ">
            {{
              dataCourse.course.teacher.map((teacher) => teacher.name).join(", ")
            }}
          </span>
          <span v-else>N/A</span>
        </p>
      </div>
    </div>
  </div>
  <hr class="border border-grey border-1 opacity-50 mb-3" />
  <div class="header-table px-4">
    <h5 class="m-0">Danh sách chương</h5>
    <div>
      <router-link :to="{ path: '/chapter-mentor-management', query: { idCourse: idCourse } }" type="button"
        class="btn btn-primary mr-3">Chương & Hỗ trợ
      </router-link>
      <router-link :to="{ path: '/add-chapter', query: { idCourse: idCourse } }" type="button"
        class="btn btn-primary mr-3">Thêm chương</router-link>
      <router-link :to="{ path: '/sort-chapter', query: { idCourse: idCourse } }" class="btn btn-primary">Sắp xếp
        chương</router-link>
    </div>
  </div>
  <Table :header="header" :data="data.chapter" :keys="keys" :actions="actions" :totalRows="totalRows" :perPage="perPage"
    @delete-item="deleteChapter" @pageChange="handlePageChange"></Table>
  <b-modal v-model="isModalVisible" title="Xác nhận xóa" ok-title="Xóa" cancel-title="Đóng" ok-variant="danger"
    @ok="handleDelete">
    <p>Bạn có chắc chắn muốn xóa chương không?</p>
  </b-modal>
</template>

<script setup>
import Table from "@/components/Tables/Table.vue";
import axios from "axios";
import { reactive, onMounted, ref } from "vue";
import { useRoute } from "vue-router";
import { toast } from "vue3-toastify";
import { useRouter } from "vue-router";

const rootAPI = process.env.VUE_APP_ROOT_API;
const route = useRoute();
const router = useRouter();
const idCourse = route.query.idCourse;

const data = reactive({
  chapter: [],
});

const isModalVisible = ref(false);
const itemToDelete = ref();

const dataCourse = reactive({
  course: {},
});

const header = ["STT", "Tên chương", "Hành động"];
const keys = ["name"];

const actions = {
  view: (item) => `/lessons?idChapter=${item.id}&idCourse=${idCourse}`,
  edit: (item) => ({
    path: `/edit-chapter/${item.id}`,
    query: { idCourse: idCourse },
  }),
  delete: (item) => `/courses/${item.id}`,
};

const currentPage = ref(1);
const perPage = ref(0);
const totalRows = ref(0);

const fetchChapter = async () => {
  try {
    const response = await axios.get(`${rootAPI}/chapters`, {
      params: {
        idCourse: idCourse,
        page: currentPage.value,
      },
    });
    data.chapter = response.data.data.items;

    console.log(response.data.data);
    perPage.value = response.data.data.pageSize;
    totalRows.value =
      response.data.data.totalPage > 0 ? response.data.data.totalPage : 1;
  } catch (error) {
    console.error(error);
  }
};

const fetchCourse = async () => {
  try {
    const response = await axios.get(`${rootAPI}/courses/${idCourse}`);
    dataCourse.course = response.data.data;
  } catch (error) {
    console.error(error);
  }
};

// Hàm xử lý xóa khóa học
const deleteChapter = (chapter) => {
  isModalVisible.value = true;
  itemToDelete.value = chapter;
};

const handleDelete = async () => {
  try {
    await axios.delete(`${rootAPI}/chapters/${itemToDelete.value.id}`);
    await fetchChapter();
    isModalVisible.value = false;
    toast.success("Xóa chương thành công");
    setTimeout(() => {
      router.replace({ path: route.path, query: { idCourse: idCourse } });
    }, 1000);
  } catch (error) {
    console.log(error);
    toast.error("Có lỗi xảy ra");
  }
};

const handlePageChange = (page) => {
  currentPage.value = page;
  fetchChapter();
};

const formatCurrency = (value, unit) => {
  if (typeof value !== "number") {
    return value;
  }
  var formatter;
  switch (unit) {
    case "USD":
      formatter = new Intl.NumberFormat("en-US", {
        style: "currency",
        currency: "USD",
      });
      break;
    case "VND":
      formatter = new Intl.NumberFormat("vi-VN", {
        style: "currency",
        currency: "VND",
      });

      break;
  }

  return formatter.format(value);
};

onMounted(async () => {
  await fetchChapter();
  await fetchCourse();
});
</script>

<style scoped>
.text-decoration-none {
  text-decoration: none;
}

i {
  font-size: 1rem;
}

img {
  border-radius: 10px;
  max-width: 80%;
  height: auto;
}

.header-table {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-table h5 {
  margin-left: 30px;
  margin-bottom: 0;
}
</style>
