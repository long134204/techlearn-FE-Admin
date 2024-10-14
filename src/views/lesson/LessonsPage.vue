<template>
  <div class="d-flex mt-3 justify-content-between align-items-center">
    <router-link :to="`/chapters?idCourse=${idCourse}`" class="text-decoration-none">
      <div class="d-flex align-items-center gap-2">
        <i class="fa-solid fa-arrow-left text-dark"></i>
        <p class="mb-0 text-dark">Danh sách các chương</p>
      </div>
    </router-link>
  </div>
  <hr class="border border-grey border-1 opacity-50" />
  <div class="px-4 d-flex justify-content-between align-items-center">
    <h4 class=" m-0" style="margin-left: 30px; margin-bottom: -20px">
      Danh sách bài học
    </h4>
    <div>
      <router-link :to="{
        path: '/add-lessons',
        query: { idChapter: idChapter, idCourse: idCourse },
      }" type="button" class="btn btn-primary  btn-sm mr-3">Thêm bài học</router-link>
      <router-link :to="{
        path: '/sort-lessons',
        query: { idChapter: idChapter, idCourse: idCourse },
      }" class="btn btn-primary  btn-sm">Sắp xếp bài học</router-link>
    </div>

  </div>
  <Table :header="header" :data="data.assignments" :keys="keys" :actions="actions" @delete-item="deleteLesson"
    :viewDetail="false" :totalRows="totalRows" :perPage="perPage" @pageChange="handlePageChange"></Table>
  <b-modal v-model="isModalVisible" title="Xác nhận xóa" ok-title="Xóa" cancel-title="Đóng" ok-variant="danger"
    @ok="handleDelete">
    <p>Bạn có chắc chắn muốn xóa bài học không?</p>
  </b-modal>
</template>

<script setup>
import axios from "axios";
import { reactive } from "vue";
import { onMounted, ref } from "vue";
import { useRoute, useRouter } from "vue-router";
import Table from "@/components/Tables/Table.vue";
import { toast } from "vue3-toastify";

const rootAPI = process.env.VUE_APP_ROOT_API;

const router = useRouter();
const route = useRoute();

const idChapter = route.query.idChapter;
const idCourse = route.query.idCourse;

const data = reactive({
  assignments: [],
});

const isModalVisible = ref(false);
const itemToDelete = ref();

const header = ["STT", "Tên Bài Học", "Hành động"];
const keys = ["title"];

const actions = {
  view: (item) => `/lessons?idChapter=${item.id}&idCourse=${idCourse}`,
  edit: (item) => ({
    path: `/lessons-update/${item.id}`,
    query: { idChapter: idChapter, idCourse: idCourse },
  }),
  delete: (item) => `/courses/${item.id}`,
};

const currentPage = ref(1);
const perPage = ref(8);
const totalRows = ref(0);

const fetchAssignments = async () => {
  try {
    const response = await axios.get(`${rootAPI}/chapters/${idChapter}/lessons`, {
      params: {
        page: currentPage.value,
        pageSize: perPage.value
      },
    });
    perPage.value = response.data.data.pageSize;
    data.assignments = response.data.data.items;
    totalRows.value = response.data.data.totalPage > 0 ? response.data.data.totalPage : 1;
    data.assignments = data.assignments.map((item) => {
      let titleUpdated = item.title;

      switch (item.type) {
        case "LECTURES":
          titleUpdated = `[Bài giảng] - ${item.title}`;
          break;
        case "READINGS":
          titleUpdated = `[Bài đọc] - ${item.title}`;
          break;
        case "EXERCISES":
          titleUpdated = `[Bài tập] - ${item.title}`;
          break;
      }

      return {
        ...item,
        title: titleUpdated,
      };
    });
  } catch (error) {
    console.log(error);
  }
};

const handlePageChange = (page) => {
  currentPage.value = page;
  fetchAssignments();
};

// xử lý xóa bài tập
const handleDelete = async () => {
  try {
    await axios.delete(`${rootAPI}/lessons/${itemToDelete.value.id}`);
    await fetchAssignments();
    isModalVisible.value = false;
    router.replace({
      path: route.path,
      query: { idCourse: idCourse, idChapter: idChapter },
    });
    toast.success("Xóa bài tập thành công");
  } catch (error) {
    console.log(error);
    toast.error("Có lỗi xảy ra");
  }
};

const deleteLesson = (lesson) => {
  isModalVisible.value = true;
  itemToDelete.value = lesson;
};

onMounted(async () => {
  await fetchAssignments();
});
</script>

<style scoped></style>
