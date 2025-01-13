<template>
    <div class="daily-container">
        <Heatmap ref="heatmap"/>
    </div>
</template>
</style>
<script lang="ts">
import { App, TFile } from 'obsidian'
import { defineComponent, ref, onMounted, defineProps, toRef, inject } from "vue";
import Heatmap from './heatmap.vue';
import { myAppInterface } from './interfaces'
export default defineComponent({
    name: "Daily",
    components: {
        Heatmap
    },
    setup(props: myAppInterface) {
        
        const heatmap = ref<InstanceType<typeof Heatmap>>()
        interface Times {
            year: number,
            month: number[]
        }
        const myApp = inject<myAppInterface>('myApp') || {
            vault: {
                getAbstractFileByPath: Function,
                read: Function
            }
        };
        let renderCanledar = ref<boolean>(false);
        let habitList = ref<any[]>([]), timesRef = ref<Times[]>([])
        onMounted(async () => {
            let files = myApp.vault.getAbstractFileByPath('01Inbox/daily').children.filter((item: TFile) => item.name != 'archives')
            // .map((item: TFile) => ({...item, todos: []}));
            let realFiles: any[] = []
            formatRealFile(files, realFiles)
            let allTodos: Object[] = []
            realFiles.forEach(async (file: any, index: number) => {
                if (file && file.path) {
                    try {
                        const fileContent = await myApp.vault.read(file);
                        const habitContent = parseTodos(getHabitContent(fileContent), file.name.replace('.md', ''), file)
                        // console.log(habitContent, 111222333)
                        // const todos = parseTodos(fileContent, file.name.replace('.md', ''));
                        // file.todos = todos || []
                        allTodos = [...allTodos, habitContent]
                        if(index == realFiles.length - 1) {
                            habitList.value = allTodos
                            renderCanledar.value = true
                            console.log(heatmap)
                            if(heatmap.value) {
                                heatmap.value.renderHeatMap(allTodos)
                            }

                        }
                    } catch (error) {
                        console.error('Error reading file:', error);
                    }
                }
            })
        })
        function getHabitContent(markdownContent: string):string {
            // 使用正则表达式匹配到habit标题内的内容
            const habitContentMatch = markdownContent.match(/## habit\n(.*?)(?=\n##|\n---|$)/s);
            if (!habitContentMatch) {
                return '';
            }

            return habitContentMatch[1].trim();
        }
        function formatRealFile(files: [], result: any[]) {
            files.forEach((item: any) => {
                if(item.children && item.children.length) {
                    formatRealFile(item.children, result)
                }else {
                    result.push(item)
                }
            })
        }
        function parseTodos(content: string, date: string, file: any) {
            const regex = /- \[([x ])\] .*/g;

            let matches = content.match(regex);
            if (matches) {
                let result = matches.map((item: string) => formatTask(item, date));
                
                return {
                    habits: result,
                    date,
                    file
                }
            } else {
                return {
                    habits: [],
                    date,
                    file
                }
            }
        }


        function formatTask(inputStr: string, date: string) {
            // 先去除行首行尾的空格
            const trimmedLine = inputStr.trim();
                // 使用正则表达式匹配并去除 "- [ ]" 或 "- [x]" 前缀，以及后续可能出现的空格
            const content = trimmedLine.replace(/^- \[\s*\]|^- \[x\]\s*/, '').trim();
            const status = trimmedLine.includes('[x]') ? 1 : 0;
            return {content, status};
        }
        let showTimeLine = ref<boolean>(false)
        function showTimeLineAction(type: boolean) {
            showTimeLine.value = type
        }

        // 获取当前日期对象
        const now = new Date();

        // 获取当前年份
        const year = now.getFullYear();

        // 获取当前月份，月份是从0开始的，所以需要加1
        const month = now.getMonth() + 1;

        console.log(`当前年份：${year}`);
        console.log(`当前月份：${month}`);

        return {
            habitList,
            renderCanledar,
            showTimeLine,
            showTimeLineAction,
            heatmap
        }
    }
})
</script>

<style scoped>
.sample-class {
    margin-left: 1em;
    color: #6C31E3;
}

.daily-container {
    width: 100%;
    height: 100%;
    display: flex;
    border: solid 1px #f0f5f9;
    padding: 10px 0;
}

.filter-box {
    width: 100%;
    height: 60px;
}

.daily-container .content {
    flex: 1;
    height: 100%;
    display: inline-block;
}
.daily-container .task{
    width: 350px;
    height: 100%;
    display: inline-block;
}


.daily-container .years-box {
    height: 100%;
    display: inline-flex;
    flex-direction: column;
    padding-left: 5px;
}

.daily-container .years-box span {
    display: inline-block;
    width: 30px;
    height: 75px;
    border-top-left-radius: 8px;
    border-bottom-left-radius: 8px;
    color: #393e46;
    text-align: center;
    word-break: break-all;
    padding: 2px 7px;
    line-height: 1;
    font-size: 16px;
    cursor: pointer;
    margin-bottom: 5px;
    margin-left: 5px;
}

.daily-container .years-box span.active {
    width: 35px;
    padding: 3px 10px;
    margin-left: 0;
}
</style>