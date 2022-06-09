<template>
  <n-data-table
    remote
    ref="haleTableRef"
    :columns="columns"
    :data="tableData"
    :paging="false"
    :loading="loading"
    :scroll-x="scrollX"
    :single-line="false"
    :style="{ height: `${tableHeight}px` }"
    flex-height
  />
</template>
<script lang="ts" setup>
import { ref, reactive, defineProps, computed, onMounted } from "vue";
import { NDataTable, DataTableColumn } from "naive-ui";
const props = defineProps<{
  tableData: any[]; //表格数据
  fixedColumn: any[]; //固定的列头
  actionColumn: any[]; //可变的列头
  tableHeight: string | number;
  isSelect?: boolean;
}>();
const scrollX = ref(0); //横向宽度
const selectColumn = {
  type: "selection",
  width: 100,
  fixed: "left",
};
//生成的列头
const columns = computed(() => {
  scrollX.value =0;
  let newColumns = [...props.fixedColumn, ...props.actionColumn];
  //如果设置了选中，则设置选中
  if (props.isSelect) newColumns.unshift(selectColumn);
  totalWidth(newColumns);
  console.log(newColumns);
  return newColumns;
});
//设置总宽度
const totalWidth = (arr) => {
  arr.forEach((v) => {
    if (v.children?.length) {
      totalWidth(v.children);
    } else {
      if (!v.width) v.width = 150;
      scrollX.value = scrollX.value + (v.width as number);
    }
  });
};
const loading = ref(false); //是否加载中
const haleTableRef = ref();
onMounted(() => {
  console.log(haleTableRef.value);
});
</script>
<style lang="less" scoped>
</style>