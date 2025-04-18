<script setup lang="ts">
import { ref, markRaw, onMounted } from "vue";
import ReCol from "@/components/ReCol";
import { useDark, randomGradient } from "./utils";
import PureTable from "./components/table/index.vue";
import { ReNormalCountTo } from "@/components/ReCountTo";
import { useRenderFlicker } from "@/components/ReFlicker";
import { barChart, lineChart, roundChart } from "./components/chart";
import Segmented, { type OptionsType } from "@/components/ReSegmented";
import { barChartData, progressData, latestNewsData } from "./data";

// import GroupLine from "@iconify-icons/ri/group-line";
// import Question from "@iconify-icons/ri/question-answer-line";
// import CheckLine from "@iconify-icons/ri/chat-check-line";
// import Smile from "@iconify-icons/ri/star-smile-line";

import { getChartData } from '@/api/common/panel'; // 请确保路径正确

/** 用户人数、评论、商品、销量 */
const chartData = ref(''); // 使用ref声明chartData

// Fetch user count from the API
const fetchPanel = async () => {
  try {
    const response = await getChartData();
    if (response.code === 200) {
      // chartData = JSON.parse(JSON.stringify(chartData));
      chartData.value = JSON.parse(response.data)
    }
  } catch (error) {
    console.error('Error fetching user count:', error);
  }
};

// 模拟从接口获取数据
onMounted(() => {
  fetchPanel();
  // chartData = 1
});

defineOptions({
  name: "Welcome"
});

const { isDark } = useDark();

let curWeek = ref(1); // 0上周、1本周
const optionsBasis: Array<OptionsType> = [
  {
    label: "上周"
  },
  {
    label: "本周"
  }
];
</script>

<template>
  <div>
    <el-row :gutter="24" justify="space-around">
      <re-col v-for="(item, index) in chartData" :key="index" v-motion class="mb-[18px]" :value="6" :md="12" :sm="12"
        :xs="24" :initial="{
          opacity: 0,
          y: 100
        }" :enter="{
  opacity: 1,
  y: 0,
  transition: {
    delay: 80 * (index + 1)
  }
}">
        <el-card class="line-card" shadow="never">
          <div class="flex justify-between">
            <span class="text-md font-medium">
              {{ item.name }}
            </span>
            <div class="w-8 h-8 flex justify-center items-center rounded-md" :style="{
              backgroundColor: isDark ? 'transparent' : item.bgColor
            }">
              <IconifyIconOnline :icon="item.icon" :color="item.color" width="18" />
            </div>
          </div>
          <div class="flex justify-between items-start mt-3">
            <div class="w-1/2">
              <ReNormalCountTo :duration="item.duration" :fontSize="'1.6em'" :startVal="100"
                :endVal="Number(item.TheValue)" />
              <p class="font-medium text-green-500">{{ item.percent }}</p>
            </div>
            <lineChart v-if="item.data.length > 1" class="!w-1/2" :color="item.color" :data="item.data" />
            <roundChart v-else class="!w-1/2" />
          </div>
        </el-card>
      </re-col>

      <re-col v-motion class="mb-[18px]" :value="18" :xs="24" :initial="{
        opacity: 0,
        y: 100
      }" :enter="{
  opacity: 1,
  y: 0,
  transition: {
    delay: 400
  }
}">
        <el-card class="bar-card" shadow="never">
          <div class="flex justify-between">
            <span class="text-md font-medium">分析概览</span>
            <Segmented v-model="curWeek" :options="optionsBasis" />
          </div>
          <div class="flex justify-between items-start mt-3">
            <barChart :requireData="barChartData[curWeek].requireData"
              :questionData="barChartData[curWeek].questionData" />
          </div>
        </el-card>
      </re-col>

      <re-col v-motion class="mb-[18px]" :value="6" :xs="24" :initial="{
        opacity: 0,
        y: 100
      }" :enter="{
  opacity: 1,
  y: 0,
  transition: {
    delay: 480
  }
}">
        <el-card shadow="never">
          <div class="flex justify-between">
            <span class="text-md font-medium">解决概率</span>
          </div>
          <div v-for="(item, index) in progressData" :key="index" :class="[
            'flex',
            'justify-between',
            'items-start',
            index === 0 ? 'mt-8' : 'mt-[2.15rem]'
          ]">
            <el-progress :text-inside="true" :percentage="item.percentage" :stroke-width="21" :color="item.color" striped
              striped-flow :duration="item.duration" />
            <span class="text-nowrap ml-2 text-text_color_regular text-sm">
              {{ item.week }}
            </span>
          </div>
        </el-card>
      </re-col>

      <re-col v-motion class="mb-[18px]" :value="18" :xs="24" :initial="{
        opacity: 0,
        y: 100
      }" :enter="{
  opacity: 1,
  y: 0,
  transition: {
    delay: 560
  }
}">
        <el-card shadow="never" class="h-[580px]">
          <div class="flex justify-between">
            <span class="text-md font-medium">数据统计</span>
          </div>
          <PureTable class="mt-3" />
        </el-card>
      </re-col>

      <re-col v-motion class="mb-[18px]" :value="6" :xs="24" :initial="{
        opacity: 0,
        y: 100
      }" :enter="{
  opacity: 1,
  y: 0,
  transition: {
    delay: 640
  }
}">
        <el-card shadow="never">
          <div class="flex justify-between">
            <span class="text-md font-medium">最新动态</span>
          </div>
          <el-scrollbar max-height="504" class="mt-3">
            <el-timeline>
              <el-timeline-item v-for="(item, index) in latestNewsData" :key="index" center placement="top" :icon="markRaw(
                useRenderFlicker({
                  background: randomGradient({
                    randomizeHue: true
                  })
                })
              )
                " :timestamp="item.date">
                <p class="text-text_color_regular text-sm">
                  {{
                    `新增 ${item.requiredNumber} 条问题，${item.resolveNumber} 条已解决`
                  }}
                </p>
              </el-timeline-item>
            </el-timeline>
          </el-scrollbar>
        </el-card>
      </re-col>
    </el-row>
  </div>
</template>

<style lang="scss" scoped>
:deep(.el-card) {
  --el-card-border-color: none;

  /* 解决概率进度条宽度 */
  .el-progress--line {
    width: 85%;
  }

  /* 解决概率进度条字体大小 */
  .el-progress-bar__innerText {
    font-size: 15px;
  }

  /* 隐藏 el-scrollbar 滚动条 */
  .el-scrollbar__bar {
    display: none;
  }

  /* el-timeline 每一项上下、左右边距 */
  .el-timeline-item {
    margin: 0 6px;
  }
}

.main-content {
  margin: 20px 20px 0 !important;
}
</style>
