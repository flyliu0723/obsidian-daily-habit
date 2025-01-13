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
                <template v-for="day in week">
                  <tiny-tooltip class="item" effect="dark" :content="day.date" placement="top">
                    <div
                      :key="day.date"
                      :class="['day', `color-${day.level}`]"
                      @click="handleDayClick(day.date, day)"
                    >
                    
                      <span v-for="item in day.colorSpan" :key="item.content"></span>
                  </div>
                  </tiny-tooltip>
                </template>
                
                
            </template>
        </template>
      </div>
    </div>
    <modal v-model="showModal" title="习惯打卡" show-footer @confirm="confirm"> 
      <tiny-checkbox-group v-model="doneHabit" type="checkbox" :options="habitOptions" @change="valueChange"></tiny-checkbox-group>
    </modal>
  </template>
  
  <script lang="ts">
  import { defineComponent, ref, onMounted, toRef, computed, inject } from 'vue';
  import { Modal, TinyTooltip, TinyCheckboxGroup } from '@opentiny/vue'
import { myAppInterface } from './interfaces'
  interface ColorSpan {
    content: string,
    status: number
  }
  interface Day {
    date: string;
    level: number;
    colorSpan: ColorSpan[],
    file: any
  }
  
  export default defineComponent({
    name: 'AnnualHeatmap',
    components: { Modal, TinyTooltip, TinyCheckboxGroup },
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
            currentWeek.push({ date: dateStr, level, file: todayHabit.file, colorSpan: (todayHabit.habits || []).filter((d: any) => d.status == 1) });
          }else {
            currentWeek.push({ date: dateStr, level, file: todayHabit ? todayHabit.file : {}, colorSpan: [] });
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
      const habitOptions = ref<[]>([]), doneHabit = ref<[]>([]), inDay = ref<any>({})
      const handleDayClick = (date: string, dayItem: any) => {
        console.log(`Clicked on ${date}`);
        console.log(dayItem)
        showModal.value = true
        inDay.value = dayItem
        habitOptions.value = dayItem.colorSpan.map((item: any) => {
          return {
            label: item.content,
            text: item.content
          }
        })
        doneHabit.value = dayItem.colorSpan.filter((item: any) => item.status == 1).map((item: any) => item.content)
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


      const TinyModal = Modal
      const showModal = ref<boolean>(false)
      function valueChange(value: any) {
        console.log(value)

      }
      const myApp = inject<myAppInterface>('myApp') || {
            vault: {
                getAbstractFileByPath: Function,
                read: Function
            }
        };
      async function confirm() {

        await updateTodosInFile(inDay.value.file, doneHabit.value)
      }

      async function updateTodosInFile(file: any, doneHabit: any[]) {
        // 获取文件内容
        const fileContent = await myApp.vault.read(file);
        console.log(doneHabit, 666555)
        // 将文件内容按行分割成数组
        const lines = fileContent.split('\n');

        // 创建一个正则表达式来匹配待办事项（假设格式为 "- [ ] 内容" 或 "- [x] 内容"）
        const todoRegex = /^(- $.?$\s*(.*))$/;
        console.log(lines, 5558888)
        // 遍历每一行并更新待办事项的状态
        let isHabit = false
        const updatedLines = lines.map((line: string) => {
          console.log(line)
          if(line == '## habit') {
            isHabit = true
          }else if(line == '## memo') {
            isHabit = false
          }
          if(!isHabit) return line
          const match = line.match(todoRegex);
          if (match) {
            console.log(match, 2222)
              const [, fullMatch, content] = match;
              // 检查内容是否在 doneHabit 中
              const isDone = doneHabit.includes(content.trim());
              return `- [${isDone ? 'x' : ' '}] ${content}`;
          }
          // 如果不是待办事项，则保留原样
          return line;
        });

        // 将更新后的行重新组合成字符串
        const updatedContent = updatedLines.join('\n');

        // 将修改后的内容写回文件
        await myApp.vault.modify(file, updatedContent);

        console.log('文件更新成功');
    }
      return {
        months,
        monthData,
        handleDayClick,
        renderHeatMap,
        showModal,
        doneHabit,
        habitOptions,
        valueChange,
        confirm
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

  .tiny-checkbox__input{
    position: relative;
    top: -5px;
  }
  </style>