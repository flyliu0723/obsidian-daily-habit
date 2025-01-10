<template>
    <div class="heatmap-container">
      <div class="month-labels">
        <div v-for="month in months" :key="month" class="month">
          {{ month }}
        </div>
      </div>
      <div class="heatmap">
        <template v-for="month in months" :key="month" class="month-heatmap">
            <template v-for="week in monthData[month]">
                <div
                    v-for="day in week"
                    :key="day.date"
                    :class="['day', `color-${day.level}`]"
                    @click="handleDayClick(day.date)"
                    >
                    <span v-for="item in day.colorSpan" :key="item.content"></span>
                  </div>
            </template>
        </template>
      </div>
    </div>
  </template>
  
  <script lang="ts">
  import { defineComponent, ref, onMounted, toRef, computed } from 'vue';
  interface ColorSpan {
    content: string,
    status: number
  }
  interface Day {
    date: string;
    level: number;
    colorSpan: ColorSpan[]
  }
  
  export default defineComponent({
    name: 'AnnualHeatmap',
    setup(props) {
      const months = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
      const monthData = ref<{ [key: string]: Day[][] }>({});
  
      const generateHeatmapData = (habitList: any) => {
        const startDate = new Date(2025, 0, 1); // 2025年1月1日
        const endDate = new Date(2025, 11, 31); // 2025年12月31日
  
        let currentDate = new Date(startDate);
        let currentWeek: Day[] = [];
        let currentMonth = startDate.getMonth();
  
        while (currentDate <= endDate) {
          const dateStr = currentDate.toISOString().split('T')[0];
          console.log(dateStr, 555)
          const level = 0
  
          if (currentDate.getMonth() !== currentMonth) {
            if (currentWeek.length > 0) {
              monthData.value[months[currentMonth]].push(currentWeek);
              currentWeek = [];
            }
            currentMonth = currentDate.getMonth();
          }
          const todayHabit = habitList.find((item: any) => item.date == dateStr)
          if(todayHabit && todayHabit.habits) {
            currentWeek.push({ date: dateStr, level, colorSpan: (todayHabit.habits || []).filter((d: any) => d.status == 1) });
          }else {
            currentWeek.push({ date: dateStr, level, colorSpan: [] });
          }
          
  
          if (currentWeek.length === 7) {
            monthData.value[months[currentMonth]].push(currentWeek);
            currentWeek = [];
          }
  
          currentDate.setDate(currentDate.getDate() + 1);
        }
  
        if (currentWeek.length > 0) {
          monthData.value[months[currentMonth]].push(currentWeek);
        }
      };
  
      const handleDayClick = (date: string) => {
        console.log(`Clicked on ${date}`);
      };
  
      onMounted(() => {
        months.forEach(month => {
          monthData.value[month] = [];
        });
      });
      function renderHeatMap(habitList: any) {
        generateHeatmapData(habitList);
      }
      const monthDataComputed = computed(() => {
        return monthData.value;
      });
      return {
        months,
        monthData,
        handleDayClick,
        renderHeatMap
      };
    },
  });
  </script>
  
  <style scoped>
  .heatmap-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .month-labels {
    display: flex;
    justify-content: space-between;
    width: 100%;
  }
  
  .month {
    margin-right: 10px;
  }
  
  .heatmap {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    flex-wrap: wrap;
    max-height: 220px;

  }
  
  .month-heatmap {
    margin-right: 20px;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    max-height: 220px;
  }
  
  .week {
    flex: 1;
    display: flex;
    flex-direction: column;
  }
  
  .day {
    width: 15px;
    height: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 2px;
    border: 1px solid #ddd;
    cursor: pointer;
  }
  .day span{
    display: inline-block;
    flex: 1;
    height: 100%;
  }
  .day span:nth-child(1) {
    background: #61D3E0;
  }
  .day span:nth-child(2) {
    background: #51C035;
  }
  .day span:nth-child(3) {
    background: #E28A6A;
  }

  
  .color-0 {
    background-color: #ebedf0;
  }
  
  .color-1 {
    background-color: #9be9a8;
  }
  
  .color-2 {
    background-color: #40c463;
  }
  
  .color-3 {
    background-color: #30a14c;
  }
  </style>