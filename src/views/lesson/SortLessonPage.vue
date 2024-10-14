<template>
    <div class="d-flex mt-3 justify-content-between align-items-center chapter-header">
        <router-link :to="{ path: '/lessons', query: { idChapter: idChapter, idCourse: idCourse } }"
            class="text-decoration-none">
            <div class="d-flex align-items-center gap-2">
                <i class="fa-solid fa-arrow-left text-dark"></i>
                <p class="mb-0 text-dark">Danh sách bài học</p>
            </div>
        </router-link>
    </div>
    <hr class="border border-grey border-1 opacity-50">
    <div class="d-flex justify-content-center flex-column align-items-center">
        <div class="d-flex justify-content-between w-100 px-4 align-items-center">
            <h4 class="title-sort m-0" style="margin-left: 30px">Sắp xếp danh sách bài học</h4>
            <button class="d-flex btn-save btn btn-warning align-self-end" @click="saveOrder">Lưu thay đổi</button>
        </div>
        <Table :header="header" :data="data.assignments" :keys="keys" :actions="actions" :isDraggable="true"
            @updateOrder="handleDragUpdate" :viewPublic="false">
        </Table>
    </div>

</template>
<script setup>
import Table from '@/components/Tables/Table.vue';
import axios from "axios";
import { reactive, onMounted } from "vue";
import { toast } from "vue3-toastify";
import { useRoute } from 'vue-router';

const rootAPI = process.env.VUE_APP_ROOT_API;
const route = useRoute();
const idChapter = route.query.idChapter;
const idCourse = route.query.idCourse;

const data = reactive({
    assignments: [],
    updatedOrder: []
});

const dataCourse = reactive({
    course: {}
});

const header = ["STT", "Tên Bài Học"];
const keys = ["title"];

function handleDragUpdate(updatedData) {
    data.updatedOrder = updatedData.map((item, index) => ({
        id: item.id,
        order: (index + 1)
    }));
}

async function saveOrder() {
    if (!data.updatedOrder.length) {
        toast.info("Không có thay đổi thứ tự nào", {
            position: "top-right",
            autoClose: 3000,
        });
        return;
    }

    try {
        await axios.patch(`${rootAPI}/lessons/update-order`, data.updatedOrder);
        toast.success("Cập nhật thứ tự thành công", {
            position: "top-right",
            autoClose: 3000,
        });
        data.updatedOrder = [];  // Clear the local updated order after saving
    } catch (error) {
        toast.error("Lỗi cập nhật thứ tự", {
            position: "top-right",
            autoClose: 3000,
        });
        console.error("Lỗi cập nhật thứ tự:", error);
    }
}

const fetchChapter = async () => {
    try {
        const response = await axios.get(`${rootAPI}/chapters/${idChapter}/lessons`);
        data.assignments = response.data.data.items;
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
        console.error(error);
    }
}

const fetchCourse = async () => {
    try {
        const response = await axios.get(`${rootAPI}/courses/${idCourse}`);
        dataCourse.course = response.data.data;
    } catch (error) {
        console.error(error);
    }
}

onMounted(async () => {
    await fetchChapter();
    await fetchCourse();
});

</script>

<style scoped></style>