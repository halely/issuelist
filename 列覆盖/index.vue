<template>
  <div class="haleWarpMain">
    <!-- 头部标题 -->
    <div class="warpHead">
      <n-h4 class="HeadTitle" prefix="bar" type="info">
        <n-text>供应商进销存查询</n-text>
      </n-h4>
    </div>
    <!-- 查询表单 -->
    <div class="haleForm">
      <n-form
        ref="formRef"
        inline
        label-width="auto"
        :model="formValue"
        label-placement="left"
        size="small"
      >
        <n-grid cols="1">
          <n-grid-item>
            <n-form-item label="仓库名称" path="storageId">
              <n-select
                v-model:value="formValue.storageId"
                multiple
                :options="storageList"
                clearable
              />
            </n-form-item>
          </n-grid-item>
        </n-grid>
        <n-grid cols="1 s:2 m:3 l:4 xl:7 2xl:6" responsive="screen" x-gap="15">
          <n-grid-item>
            <n-form-item label="状态" path="status">
              <n-select
                v-model:value="formValue.status"
                :options="styleStatus"
              />
            </n-form-item>
          </n-grid-item>

          <n-grid-item>
            <n-form-item label="供应商" path="supplierId">
              <n-select
                v-model:value="formValue.supplierId"
                :options="supplierList"
                clearable
                filterable
                :consistent-menu-width="false"
              />
            </n-form-item>
          </n-grid-item>
          <n-grid-item>
            <n-form-item label="商品条码" path="productId">
              <n-input
                v-model:value="formValue.productId"
                clearable
                placeholder="输入商品条码"
              />
            </n-form-item>
          </n-grid-item>
          <n-grid-item>
            <n-form-item label="商品名称" path="productName">
              <n-input
                v-model:value="formValue.productName"
                clearable
                placeholder="输入商品名称"
              />
            </n-form-item>
          </n-grid-item>
          <n-grid-item span="2">
            <n-form-item label="采购时间" path="createdDate">
              <n-date-picker
                format="yyyy-MM-dd"
                clearable
                v-model:formatted-value="formValue.beginTime"
                type="date"
                :is-date-disabled="startDisabled"
              ></n-date-picker
              >&nbsp;-&nbsp;
              <n-date-picker
                format="yyyy-MM-dd"
                clearable
                v-model:formatted-value="formValue.endTime"
                type="date"
                :is-date-disabled="endDisabled"
              ></n-date-picker>
            </n-form-item>
          </n-grid-item>
          <n-grid-item suffix>
            <n-form-item class="formButtonBox">
              <n-space justify="end">
                <n-button
                  type="info"
                  attr-type="button"
                  @click="searchEvent"
                  :disabled="loading"
                  >查询</n-button
                >
                <n-button attr-type="button" @click="resetForm">重置</n-button>
              </n-space>
            </n-form-item>
          </n-grid-item>
        </n-grid>
      </n-form>
    </div>
    <!-- 表格模块 -->
    <div class="haleTableBox">
      <div class="tableTitlebox">
        <div class="tableTitleMain">
          <n-h4 type="info">
            <n-text>商品采购列表</n-text>
          </n-h4>
          <p class="unit">（金额：元）</p>
        </div>
        <div class="operationButBox"></div>
      </div>
      <div class="tablebody" ref="tablebodyRef">
        <HaleTable
          :tableHeight="tableHeight"
          :tableData="tableData"
          :fixedColumn="fixedColumn"
          :actionColumn="actionColumn"
          isSelect
        />
      </div>
    </div>
  </div>
</template>
<script lang="ts" setup>
import {
  reactive,
  ref,
  nextTick,
  h,
  onMounted,
  unref,
  Component,
  computed,
} from "vue";
import {
  NH4,
  NText,
  NForm,
  NFormItem,
  NInput,
  NButton,
  NSelect,
  FormInst,
  NGrid,
  NGridItem,
  NDatePicker,
  NSpace,
} from "naive-ui";
import moment from "moment"; //引入时间控件
import elementResizeDetectorMaker from "element-resize-detector";

// 引入子组件
import HaleTable from "@/components/HaleTable.vue";

const erdm = elementResizeDetectorMaker(); //为了监听元素高度实例
const formRef = ref<FormInst | null>(null);
//默认表单数据
const defaultValueRef = () => ({
  productId: "",
  status: "",
  storageId: [],
  supplierId: null,
  productName: "",
  beginTime: moment().subtract(1, "months").format("YYYY-MM-DD"),
  endTime: moment().format("YYYY-MM-DD"),
  sortColumn: "",
  sortRule: "",
});
//构建响应式表单数据
let formValue = reactive(defaultValueRef());
//重置
function resetForm() {
  formRef.value?.restoreValidation();
  formValue = Object.assign(unref(formValue), defaultValueRef());
  nextTick(() => {
    searchEvent();
  });
}
const loading = ref<boolean>(false); //查询是否不能点击
const searchEvent = () => {};
//仓库名称
const storageList = [];
//状态
const styleStatus = [];
//供应商
const supplierList = [];

//时间范围控制
//开始时间限制
const startDisabled = (ts: number) => {
  if (formValue.endTime) {
    return new Date(ts).getTime() > new Date(formValue.endTime).getTime();
  }
  return new Date(ts).getTime() > new Date().getTime();
};
const endDisabled = (ts: number) => {
  if (formValue.beginTime) {
    return (
      new Date(ts).getTime() <
        new Date(formValue.beginTime).getTime() - 86400000 ||
      new Date(ts).getTime() > new Date().getTime()
    );
  }
  return new Date(ts).getTime() > new Date().getTime();
};

/* 监听高度变化设置表格高度 */
const tablebodyRef = ref<HTMLDivElement | null>(null);
const tableHeight = ref<number>(200);
const watchHeight = () => {
  erdm.listenTo(tablebodyRef.value as HTMLDivElement, (element: any) => {
    nextTick(() => {
      tableHeight.value = element.offsetHeight; //48是列头的高度
    });
  });
};

function createData() {
  return Array.from({ length: 50 }).map((_, i) => {
    return {
      key: i,
      wareHouseName: `name_${i}`,
      beginNumber: `beginNumber_${i}`,
      beginAmount: `beginAmount_${i}`,
      beginTotalPretaxCost: `beginTotalPretaxCost_${i}`,
      beginTax: `beginTax_${i}`,
      beginPrice: `beginPrice_${i}`,
      semesterNumber: `semesterNumber_${i}`,
      semesterAmount: `semesterAmount_${i}`,
      semesterTotalPretaxCost: `semesterTotalPretaxCost_${i}`,
      semesterTax: `semesterTax_${i}`,
      semesterPrice: `semesterPrice_${i}`,
      inStorageNumber: `inStorageNumber_${i}`,
      inStorageAmount: `inStorageAmount_${i}`,
      inStorageTotalPretaxCost: `inStorageTotalPretaxCost_${i}`,
      inStorageTax: `inStorageTax_${i}`,
      inStoragePrice: `inStoragePrice_${i}`,
      outStorageNumber: `outStorageNumber_${i}`,
      outStorageAmount: `outStorageAmount_${i}`,
      outStorageTotalPretaxCost: `outStorageTotalPretaxCost_${i}`,
      outStorageTax: `outStorageTax_${i}`,
      outStoragePrice: `outStoragePrice_${i}`,
    };
  });
}
//表格数据
const tableData = ref(createData());
const fixedColumn = [
  {
    title: "仓库名称",
    fixed: "left",
    key: "wareHouseName",
  }
];
const actionColumn = [
  {
    title: "期初",
    key: "begin",
    align: "center",
    children: [
      { title: "数量", key: "beginNumber" },
      { title: "成本金额", key: "beginAmount" },
      { title: "不含税金额", key: "beginTotalPretaxCost" },
      { title: "税额", key: "beginTax" },
      { title: "售价金额", key: "beginPrice" },
    ],
  },
  {
    title: "期末",
    key: "semester",
    align: "center",
    children: [
      { title: "数量", key: "semesterNumber" },
      { title: "成本金额", key: "semesterAmount" },
      { title: "不含税金额", key: "semesterTotalPretaxCost" },
      { title: "税额", key: "semesterTax" },
      { title: "售价金额", key: "semesterPrice" },
    ],
  },
  {
    title: "入库",
    key: "inStorage",
    align: "center",
    children: [
      { title: "数量", key: "inStorageNumber" },
      { title: "成本金额", key: "inStorageAmount" },
      { title: "不含税金额", key: "inStorageTotalPretaxCost" },
      { title: "税额", key: "inStorageTax" },
      { title: "售价金额", key: "inStoragePrice" },
    ],
  },
  {
    title: "出库",
    key: "outStorage",
    align: "center",
    children: [
      { title: "数量", key: "outStorageNumber" },
      { title: "成本金额", key: "outStorageAmount" },
      { title: "不含税金额", key: "outStorageTotalPretaxCost" },
      { title: "税额", key: "outStorageTax" },
      { title: "售价金额", key: "outStoragePrice" },
    ],
  },
];
onMounted(async () => {
  watchHeight();
});
</script>
<style lang="less" scoped>
</style>