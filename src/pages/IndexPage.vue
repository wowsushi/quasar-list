<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input
          v-model="tempData.name"
          label="姓名"
          :rules="[(val) => val.length > 0]"
        />
        <q-input
          v-model="tempData.age"
          label="年齡"
          :rules="[(val) => val.length > 0 && /^\d+$/g.test(val)]"
        />
        <q-btn color="primary" class="q-mt-md" @click="handleAddNew"
          >新增</q-btn
        >
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { config } from 'process';
import { QTableProps, useQuasar } from 'quasar';
import { onMounted, ref } from 'vue';
const $q = useQuasar();

interface btnType {
  label: string;
  icon: string;
  status: string;
}
interface blockData {
  name: string;
  age: number;
  creatorId: string;
  id: string;
}
const BASE_URL = 'https://demo.mercuryfire.com.tw:49110';
const axiosInstance = axios.create({
  baseURL: BASE_URL,
  timeout: 60 * 1000,
});
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

function doError(message: string) {
  $q.dialog({
    title: '發生錯誤',
    message,
  });
}
async function fetchBlockData() {
  try {
    const result = await axiosInstance.get('/crudTest/a');
    blockData.value = result.data.result;
  } catch (err) {
    doError(err.message);
  }
}
async function addBlockData(data: tempData) {
  try {
    await axiosInstance.post('/crudTest', {
      name: data.name,
      age: +data.age,
    });
  } catch (err) {
    doError(err.message);
  }
}
async function updateBlockData(data: tempData & { id: string }) {
  try {
    await axiosInstance.patch('/crudTest', {
      id: data.id,
      name: data.name,
      age: +data.age,
    });
  } catch (err) {
    doError(err.message);
  }
}
async function deleteBlockData(id: string) {
  try {
    await axiosInstance.delete(`/crudTest/${id}`);
  } catch (err) {
    doError(err.message);
  }
}

onMounted(() => {
  fetchBlockData();
});

type tempData = {
  name: string;
  age: string;
};
const tempData = ref<tempData>({
  name: '',
  age: '',
});
async function handleAddNew() {
  await addBlockData(tempData.value);
  fetchBlockData();
  tempData.value = {
    name: '',
    age: '',
  };
}

function confirmCancel(data: blockData) {
  $q.dialog({
    title: '提示',
    message: '是否確定刪除該筆資料？',
    cancel: '取消',
    persistent: true,
    ok: '確定',
  }).onOk(async () => {
    await deleteBlockData(data.id);
    fetchBlockData();
  });
}
async function handleClickOption(btn: btnType, data: blockData) {
  switch (btn.status) {
    case 'edit': {
      await updateBlockData({
        id: data.id,
        ...tempData.value,
      });
      fetchBlockData();
      break;
    }
    case 'delete': {
      confirmCancel(data);
      break;
    }
  }
}
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
