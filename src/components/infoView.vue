<template>
  <div>
    <el-row>
      <el-col :span="1">
        <h3>导入数据</h3>
      </el-col>
      <el-col :span="3" style="margin-top: 20px">
        <input type="file" @change="handleFileUpload" accept=".xlsx,.xls" />
      </el-col>
    </el-row>
    <el-row type="flex" style="margin-top: 20px">
      <el-input
          v-model="selectionQuery.id"
          style="width: 240px"
          size="large"
          placeholder="请输入想要搜索的ID号"
      />

      <el-button type="primary" @click="handleSelectionFilter" @sort-change="handleSortChange" style="margin-left: 10px;">搜索</el-button>
    </el-row>
    <div style="border: 1px solid #e6e6e6; margin-top: 20px">
      <el-row justify="center" style="margin-top: 10px; margin-left: 10px">
        <el-col :span="8">
          <span>baseMean:&nbsp;</span>
          <el-input-number v-model="baseMeanLeft" :controls="false" style="max-width: 80px"/>
          --
          <el-input-number v-model="baseMeanRight" :controls="false" style="max-width: 80px"/>
        </el-col>
        <el-col :span="8">
          <span>log2FoldChange:&nbsp;</span>
          <el-input-number v-model="log2FoldChangeLeft" :controls="false" style="max-width: 80px"/>
          --
          <el-input-number v-model="log2FoldChangeRight" :controls="false" style="max-width: 80px"/>
        </el-col>
        <el-col :span="8">
          <span>lfcSE:&nbsp;</span>
          <el-input-number v-model="lfcSELeft" :controls="false" style="max-width: 80px"/>
          --
          <el-input-number v-model="lfcSERight" :controls="false" style="max-width: 80px"/>
        </el-col>
      </el-row>

      <el-row style="margin-top: 20px; margin-left: 10px;">
        <el-col :span="8">
          <span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;stat:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>
          <el-input-number v-model="statLeft" :controls="false" style="max-width: 80px;"/>
          --
          <el-input-number v-model="statRight" :controls="false" style="max-width: 80px"/>
        </el-col>
        <el-col :span="8">
          <span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pValue:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>
          <el-input-number v-model="pValueLeft" :controls="false" style="max-width: 80px"/>
          --
          <el-input-number v-model="pValueRight" :controls="false" style="max-width: 80px"/>
        </el-col>
        <el-col :span="8">
          <span>pAdj:&nbsp;</span>
          <el-input-number v-model="pAdjLeft" :controls="false" style="max-width: 80px"/>
          --
          <el-input-number v-model="pAdjRight" :controls="false" style="max-width: 80px"/>
        </el-col>
      </el-row>
      <el-row justify="end" style="margin-top: 20px; margin-bottom: 10px;">
        <el-col :span="3">
          <el-text>*筛选框内输入想要筛选的数值范围</el-text>
        </el-col>
        <el-col :span="2">
          <el-button type="primary" @click="clearFilters">清空</el-button>
          <el-button type="primary" @click="applyFilters">筛选</el-button>
        </el-col>
      </el-row>
    </div>

    <el-table :data="currentPageData" stripe style="width: 100%; margin-top: 10px">
      <el-table-column prop="id" label="ID"></el-table-column>
      <el-table-column prop="baseMean" label="baseMean" sortable></el-table-column>
      <el-table-column prop="log2FoldChange" label="log2FoldChange" sortable></el-table-column>
      <el-table-column prop="lfcSE" label="lfcSE" sortable></el-table-column>
      <el-table-column prop="stat" label="stat" sortable></el-table-column>
      <el-table-column prop="pValue" label="pValue" sortable></el-table-column>
      <el-table-column prop="pAdj" label="pAdj" sortable></el-table-column>
    </el-table>

    <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :page-sizes="[10, 20, 50]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="totalItems"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import * as XLSX from 'xlsx';

interface Information {
  id: string;
  baseMean: number;
  log2FoldChange: number;
  lfcSE: number;
  stat: number;
  pValue: number;
  pAdj: number;
}

const selectionQuery = ref({
  id: ''
});
let filterList = ref([] as Information[]);

const informationList = ref<Information[]>([]);

const pageSize = ref(10);
const currentPage = ref(1);
const totalItems = ref(0);

let currentPageData = ref<Information[]>([]);

const baseMeanRight = ref(Number.POSITIVE_INFINITY);
const baseMeanLeft = ref(Number.NEGATIVE_INFINITY);
const log2FoldChangeRight = ref(Number.POSITIVE_INFINITY);
const log2FoldChangeLeft = ref(Number.NEGATIVE_INFINITY);
const lfcSERight = ref(Number.POSITIVE_INFINITY);
const lfcSELeft = ref(Number.NEGATIVE_INFINITY);
const statRight = ref(Number.POSITIVE_INFINITY);
const statLeft = ref(Number.NEGATIVE_INFINITY);
const pValueRight = ref(Number.POSITIVE_INFINITY);
const pValueLeft = ref(Number.NEGATIVE_INFINITY);
const pAdjRight = ref(Number.POSITIVE_INFINITY);
const pAdjLeft = ref(Number.NEGATIVE_INFINITY);

const handleFileUpload = (event: Event) => {
  const file = (event.target as HTMLInputElement).files![0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e: ProgressEvent<FileReader>) => {
      const data = new Uint8Array(e.target!.result as ArrayBuffer);
      const workbook = XLSX.read(data, { type: 'array' });
      const sheet1 = workbook.SheetNames[0];
      const worksheet = workbook.Sheets[sheet1];

      const excelData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

      informationList.value = (excelData as Array<Array<string | number | null>>)
          .slice(1)
          .filter(row => row.some(cell => cell!== null && cell!== ''))
          .map((row: any[]) => ({
        id: row[0]?.toString().replace('NA', ''),
        baseMean: parseFloat(row[1]) || 0,
        log2FoldChange: parseFloat(row[2]) || 0,
        lfcSE: parseFloat(row[3]) || 0,
        stat: parseFloat(row[4]) || 0,
        pValue: parseFloat(row[5]) || 0,
        pAdj: parseFloat(row[6]) || 0,
      }));

      filterList.value = informationList.value;
      totalItems.value = informationList.value.length;
      currentPage.value = 1;
      updateCurrentPageData();
    };
    reader.readAsArrayBuffer(file);
  }
}

const handleSelectionFilter = () => {
  filterList.value = informationList.value.filter((item) => {
    return item.id.includes(selectionQuery.value.id);
  });
  totalItems.value = filterList.value.length;
  currentPage.value = 1;
  updateCurrentPageData();
};

const handleSizeChange = (newSize: number) => {
  pageSize.value = newSize;
  currentPage.value = 1;
  updateCurrentPageData();
};

const handleCurrentChange = (newPage: number) => {
  currentPage.value = newPage;
  updateCurrentPageData();
};

const updateCurrentPageData = () => {
  const start = (currentPage.value - 1) * pageSize.value;
  const end = start + pageSize.value;
  currentPageData.value = filterList.value.slice(start, end);
};

const handleSortChange = (column: any) => {
  const { prop, order } = column;
  if (order === 'ascending') {
    filterList.value.sort((a, b) => a[prop] - b[prop]);
  } else if (order === 'descending') {
    filterList.value.sort((a, b) => b[prop] - a[prop]);
  }
  totalItems.value = filterList.value.length;
  currentPage.value = 1;
  updateCurrentPageData();
};

const clearFilters = () => {
  baseMeanRight.value = Number.POSITIVE_INFINITY;
  baseMeanLeft.value = Number.NEGATIVE_INFINITY;
  log2FoldChangeRight.value = Number.POSITIVE_INFINITY;
  log2FoldChangeLeft.value = Number.NEGATIVE_INFINITY;
  lfcSERight.value = Number.POSITIVE_INFINITY;
  lfcSELeft.value = Number.NEGATIVE_INFINITY;
  statRight.value = Number.POSITIVE_INFINITY;
  statLeft.value = Number.NEGATIVE_INFINITY;
  pValueRight.value = Number.POSITIVE_INFINITY;
  pValueLeft.value = Number.NEGATIVE_INFINITY;
  pAdjRight.value = Number.POSITIVE_INFINITY;
  pAdjLeft.value = Number.NEGATIVE_INFINITY;
}

const applyFilters = () => {
  console.log(baseMeanLeft.value, baseMeanRight.value);
  filterList.value = informationList.value.filter(item => {
    return (
        (item.baseMean <= baseMeanRight.value) &&
        (item.baseMean >= baseMeanLeft.value) &&
        (item.log2FoldChange <= log2FoldChangeRight.value) &&
        (item.log2FoldChange >= log2FoldChangeLeft.value) &&
        (item.lfcSE <= lfcSERight.value) &&
        (item.lfcSE >= lfcSELeft.value) &&
        (item.stat <= statRight.value) &&
        (item.stat >= statLeft.value) &&
        (item.pValue <= pValueRight.value) &&
        (item.pValue >= pValueLeft.value) &&
        (item.pAdj <= pAdjRight.value) &&
        (item.pAdj >= pAdjLeft.value)
    );
  });
  totalItems.value = filterList.value.length;
  currentPage.value = 1;
  updateCurrentPageData();
};

</script>

<style scoped>
.search-row {
  margin-bottom: 20px;
}

.el-select {
  width: 100%;
}

strong {
  font-size: 16px;
  color: #575e64;
}
</style>