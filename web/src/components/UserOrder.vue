<template>
  <div class="user-bill" v-loading="loading" element-loading-background="rgba(255,255,255,.3)">
    <el-row v-if="items.length > 0">
      <el-table :data="items" :row-key="(row) => row.id" table-layout="auto" border>
        <el-table-column prop="order_no" label="订单号">
          <template #default="scope">
            <span>{{ scope.row.order_no }}</span>
            <el-icon class="copy-order-no" :data-clipboard-text="scope.row.order_no">
              <DocumentCopy />
            </el-icon>
          </template>
        </el-table-column>
        <el-table-column prop="subject" label="产品名称" />
        <el-table-column prop="amount" label="订单金额" />
        <el-table-column label="订单算力">
          <template #default="scope">
            <span>{{ scope.row.remark?.power }}</span>
          </template>
        </el-table-column>
        <el-table-column prop="pay_method" label="支付渠道" />
        <el-table-column prop="pay_name" label="支付名称" />
        <el-table-column label="支付时间">
          <template #default="scope">
            <span v-if="scope.row['pay_time']">{{ dateFormat(scope.row["pay_time"]) }}</span>
            <el-tag v-else>未支付</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-row>
    <el-empty :image-size="100" v-else :image="nodata" description="暂无数据" />
    <div class="pagination pb-5">
      <el-pagination
        v-if="total > 0"
        background
        layout="total,prev, pager, next"
        :hide-on-single-page="true"
        v-model:current-page="page"
        v-model:page-size="pageSize"
        @current-change="fetchData()"
        style="--el-pagination-button-bg-color: rgba(86, 86, 95, 0.2)"
        :total="total"
      />
    </div>
  </div>
</template>

<script setup>
import nodata from "@/assets/img/no-data.png";

import { onMounted, ref } from "vue";
import { httpGet } from "@/utils/http";
import { ElMessage } from "element-plus";
import { dateFormat } from "@/utils/libs";
import { DocumentCopy } from "@element-plus/icons-vue";
import Clipboard from "clipboard";

const items = ref([]);
const total = ref(0);
const page = ref(1);
const pageSize = ref(12);
const loading = ref(true);

onMounted(() => {
  fetchData();
  const clipboard = new Clipboard(".copy-order-no");
  clipboard.on("success", () => {
    ElMessage.success("复制成功！");
  });

  clipboard.on("error", () => {
    ElMessage.error("复制失败！");
  });
});

// 获取数据
const fetchData = () => {
  httpGet("/api/order/list", { page: page.value, page_size: pageSize.value })
    .then((res) => {
      if (res.data) {
        items.value = res.data.items;
        total.value = res.data.total;
        page.value = res.data.page;
        pageSize.value = res.data.page_size;
      }
      loading.value = false;
    })
    .catch((e) => {
      ElMessage.error("获取数据失败：" + e.message);
    });
};
</script>

<style scoped lang="stylus">
.user-bill {
  background-color: var(--chat-bg);

  .pagination {
    margin: 20px 0 0 0;
    display: flex;
    justify-content: center;
    width: 100%;
  }

  .copy-order-no {
    cursor pointer
    position relative
    left 6px
    top 2px
    color #20a0ff
  }
}
</style>
