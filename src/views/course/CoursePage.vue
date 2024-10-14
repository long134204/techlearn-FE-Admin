<template>
  <div class="d-flex justify-content-between align-items-center my-4 container px-4">
    <h4 class="course-list-title">Danh sách khóa học</h4>
    <button class="btn btn-primary create-course-btn align-items-center" @click="createCourse">
      Thêm mới
    </button>
  </div>
  <Table :header="header" :data="data.courses" :keys="keys" :actions="actions" :totalRows="totalRows" :perPage="perPage"
    @deleteItem="deleteCourse" @pageChange="handlePageChange"></Table>
  <b-modal v-model="isModalVisible" title="Xác nhận xóa" ok-title="Xóa" cancel-title="Đóng" ok-variant="danger"
    @ok="handleDelete">
    <p>Bạn có chắc chắn xóa khóa học không?</p>
  </b-modal>
</template>

<script setup>
import { useRouter } from "vue-router";
import { reactive, onMounted, ref } from "vue";
import axios from "axios";
import Table from "@/components/Tables/Table.vue";
import { toast } from "vue3-toastify";

const router = useRouter();
const rootAPI = process.env.VUE_APP_ROOT_API;

const isModalVisible = ref(false);
const itemToDelete = ref();

const data = reactive({
  courses: [],
});
const unit = ref();

const header = ["STT", "Tên khóa học", "Giá tiền", "Đơn vị", "Hành động"];
const keys = ["name", "price", "currencyUnit"];

const actions = {
  view: (item) => `/chapters?idCourse=${item.id}`,
  edit: (item) => `/courses-update/${item.id}`,
  delete: (item) => `/courses/${item.id}`,
};

const currentPage = ref(1);
const perPage = ref(9);
const totalRows = ref(0);

const fetchCourses = async () => {
  try {
    const response = await axios.get(`${rootAPI}/courses`, {
      params: {
        page: currentPage.value,
        pageSize: perPage.value
      },
    });
    data.courses = response.data.data.items.map(course => {
      return {
        ...course,
        price: formatCurrency(course.price, course.currencyUnit)
      };
    });

    perPage.value = response.data.data.pageSize;
    totalRows.value = response.data.data.totalPage > 0 ? response.data.data.totalPage : 1;
  } catch (error) {
    console.error("Error fetching courses", error);
  }
};

const handlePageChange = (page) => {
  currentPage.value = page;
  fetchCourses();
};

const deleteCourse = (course) => {
  isModalVisible.value = true;
  itemToDelete.value = course;
};

const handleDelete = async () => {
  try {
    await axios.delete(`${rootAPI}/courses/${itemToDelete.value.id}`);
    await fetchCourses();
    isModalVisible.value = false;
    toast.success("Xóa khóa học thành công");
  } catch (error) {
    console.log(error);
    toast.error("Có lỗi xảy ra");
  }
};

const createCourse = () => {
  router.push("/courses-create");
};

function formatCurrency(value, unit) {
  if (typeof value !== "number") {
    return value;
  }
  var formatter;
  switch (unit) {
    case "USD":
      formatter = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD'
      });
      return formatter.format(value).replace('$', '');
    case "VND":
      formatter = new Intl.NumberFormat('vi-VN', {
        style: 'decimal',
        currency: 'VND'
      });
      return formatter.format(value);
  }
}


onMounted(async () => {
  await fetchCourses()
});
</script>

<style scoped>
/* .course-list-title {
  margin-left: 20px !important;
  font-weight: 600;
  font-size: 25px;
  margin: 0;
}

.create-course-btn {
  margin-right: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
} */
</style>
