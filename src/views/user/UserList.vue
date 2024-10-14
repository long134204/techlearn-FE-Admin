<template>

    <div class="d-flex justify-content-between align-items-center my-4 container">
    <h3 class="course-list-title">Danh sách người dùng</h3>
    <button class="btn btn-primary create-course-btn align-items-center" @click="createUser">
      Thêm mới
    </button>
  </div>
  <Table :header="header" :data="listUser.data" :keys="keys" :actions="actions" :totalRows="totalRows" :perPage="perPage"
    @deleteItem="deleteUser" @pageChange="handlePageChange" :isUserPage=true></Table>
  <b-modal v-model="isModalVisible" title="Xác nhận xóa" ok-title="Xóa" cancel-title="Đóng" ok-variant="danger"
    @ok="handleDelete">
    <p>Bạn có chắc chắn xóa khóa học không?</p>
  </b-modal>
</template>
<script setup>

import axios from 'axios';
import { onMounted, reactive,ref } from 'vue';
import { toast } from "vue3-toastify";
import Table from "@/components/Tables/Table.vue";
import router from '@/router';

const rootAPI = process.env.VUE_APP_ROOT_API;
const header = ["STT","Ảnh", "Họ và tên", "Email", "vai trò", "Hành động"];
const keys = ["avatar", "fullName", "email", "roles"];
const actions = {
  edit: (item) => `/users-update/${item.id}`,
  delete: (item) => `/users/${item.id}`,
};
const isModalVisible = ref(false);
const itemToDelete = ref();
const currentPage = ref(1);
const perPage = ref(9);
const totalRows = ref(0);

const listUser = reactive({
    data:[]
})

const deleteUser = (user) => {
  isModalVisible.value = true;
  itemToDelete.value = user;
};

const getAllUser = async () => {
    try{
        const res = await axios.get(`${rootAPI}/users`);
        listUser.data = res.data.data.items;
        console.log(res.data.data.items);
        perPage.value = res.data.data.pageSize;
        totalRows.value = res.data.data.totalPage > 0 ? res.data.data.totalPage : 1;
    }catch(err) {
        toast.error("Lấy danh sách người dùng thất bại");
    }
}

const handlePageChange = (page) => {
  currentPage.value = page;
  getAllUser();
};

const createUser = () => {
    router.push("/create-user");
}

const handleDelete = async () => {
  try {
    await axios.delete(`${rootAPI}/users/${itemToDelete.value.id}`);
    await getAllUser();
    isModalVisible.value = false;
    toast.success("Xóa khóa học thành công");
  } catch (error) {
    console.log(error);
    toast.error("Có lỗi xảy ra");
  }
}

onMounted(async() => {
await getAllUser();
})

</script>
<style scoped>
.course-list-title {
  font-weight: bold;
}

.card-body {
    background-color: white;
    border-radius: 8px;
}

h2 {
    color: #5a5a5a;
    font-weight: 600;
    margin-bottom: 1.5rem;
}

.row.justify-content-between {
    margin-bottom: 1.5rem;
}

.form-control {
    border: 1px solid #e0e0e0;
    border-radius: 20px;
    padding: 0.5rem 1rem;
    width: 100%;
    font-size: 0.9rem;
}

.user-list-files a {
    background-color: #0B5ED7;
    color: white;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    font-size: 0.9rem;
    text-decoration: none;
    transition: background-color 0.3s ease;
    margin-left: 0.5rem;
}

.user-list-files a:hover {
    background-color: #0B5ED7;
}

.table {
    border-collapse: separate;
    border-spacing: 0 0.5rem;
}

thead {
    background-color: #f8fafc;
}

thead th {
    border: none;
    font-weight: 600;
    color: #64748b;
    padding: 1rem;
    font-size: 0.9rem;
    text-transform: uppercase;
}

tbody tr {
    background-color: white;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
    transition: all 0.3s ease;
}

tbody tr:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

tbody td {
    vertical-align: middle;
    padding: 1rem;
    border: none;
    font-size: 0.9rem;
}

td, th {
    white-space: nowrap;
    overflow: hidden; 
    text-overflow: ellipsis; 
    max-width: 150px;
}

td:nth-child(2), 
td:nth-child(4), 
td:nth-child(5), 
td:nth-child(7)  {
    max-width: 200px; 
}

.avatar-40 {
    width: 40px;
    height: 40px;
    object-fit: cover;
}

.badge {
    padding: 0.35rem 0.7rem;
    font-size: 0.75rem;
    border-radius: 20px;
    font-weight: 500;
}


.badge.bg-primary {
    color: white;
}

.bg-edit{
    background-color: #efb400 !important;
}

.bg-delete{
    background-color: #df3f4f !important;
}

.badge.bg-warning-light {
    background-color: #fef3c7;
    color: #d97706;
}

.badge.bg-danger-light {
    background-color: #fee2e2;
    color: #dc2626;
}

.list-user-action {
    display: flex;
    gap: 0.5rem;
}

.btn-sm {
    padding: 0.25rem;
    font-size: 0.875rem;
    border-radius: 50%;
    width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background-color 0.3s ease;
}

.btn-sm:hover {
    background-color: #0B5ED7;
}

.btn-sm i {
    font-size: 1rem;
}

@media (max-width: 768px) {
    td, th {
        max-width: 100px; 
    }
}

.table-responsive {
    overflow-x: auto;
}

.table-responsive::after {
    content: '';
    position: absolute;
    bottom: 0;
    right: 0;
    width: 50px;
    height: 2px;
    background: linear-gradient(to right, transparent, #38bdf8);
    opacity: 0.5;
}

.table-responsive {
    -ms-overflow-style: none;  /* IE and Edge */
    scrollbar-width: none;  /* Firefox */
}

.table-responsive::-webkit-scrollbar {
    display: none;
}
</style>