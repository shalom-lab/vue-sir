<template>
    <div class="visualization-section">
        <v-chart class="chart" :option="option" autoresize @globalout="handleGlobalOut" />
    </div>
</template>

<script setup>
import { use } from "echarts/core"
import { CanvasRenderer } from "echarts/renderers"
import { LineChart } from "echarts/charts"
import {
    TitleComponent,
    TooltipComponent,
    LegendComponent,
    GridComponent
} from "echarts/components"
import VChart from "vue-echarts"
import { computed } from "vue"

// 注册必须的组件
use([
    CanvasRenderer,
    LineChart,
    TitleComponent,
    TooltipComponent,
    LegendComponent,
    GridComponent
])

const props = defineProps({
    data: {
        type: Array,
        default: () => []
    }
})

const emit = defineEmits(['hover-data-change'])

// 计算最大天数和最大人口
const maxDays = computed(() => {
    return props.data.length ? props.data[props.data.length - 1][0] : 100
})

const maxPopulation = computed(() => {
    if (!props.data.length) return 1000
    return Math.max(...props.data.map(([_, S, I, R]) => S + I + R))
})

// 图表配置
const option = computed(() => ({
    title: {
        text: 'SIR 模型演化曲线',
        left: 'center',
        top: 10
    },
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            type: 'line',
            animation: false
        },
        formatter: (params) => {
            if (params && params.length > 0) {
                const day = params[0].data[0]
                const S = Math.round(params[0].data[1])
                const I = Math.round(params[1].data[1])
                const R = Math.round(params[2].data[1])
                const total = S + I + R

                // 发出事件通知父组件当前悬停的数据
                emit('hover-data-change', { day, S, I, R })

                return `
                    <div style="font-weight: bold; margin-bottom: 8px">第 ${day} 天</div>
                    <div style="display: flex; justify-content: space-between; margin: 4px 0">
                        <span style="color: #5470c6">易感人群 (S):</span>
                        <span style="font-weight: 500">${S} (${(S/total*100).toFixed(1)}%)</span>
                    </div>
                    <div style="display: flex; justify-content: space-between; margin: 4px 0">
                        <span style="color: #ee6666">感染人群 (I):</span>
                        <span style="font-weight: 500">${I} (${(I/total*100).toFixed(1)}%)</span>
                    </div>
                    <div style="display: flex; justify-content: space-between; margin: 4px 0">
                        <span style="color: #91cc75">恢复人群 (R):</span>
                        <span style="font-weight: 500">${R} (${(R/total*100).toFixed(1)}%)</span>
                    </div>
                `
            }
            return ''
        }
    },
    legend: {
        data: ['易感人群 (S)', '感染人群 (I)', '恢复人群 (R)'],
        top: 40
    },
    grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        top: '80px',
        containLabel: true
    },
    xAxis: {
        type: 'value',
        name: '时间 (天)',
        nameLocation: 'middle',
        nameGap: 30,
        min: 0,
        max: maxDays.value
    },
    yAxis: {
        type: 'value',
        name: '人数',
        nameLocation: 'middle',
        nameGap: 40,
        min: 0,
        max: maxPopulation.value.toFixed(0)
    },
    series: [
        {
            name: '易感人群 (S)',
            type: 'line',
            smooth: true,
            data: props.data.map(item => [item[0], item[1]]),
            itemStyle: { color: '#5470c6' }
        },
        {
            name: '感染人群 (I)',
            type: 'line',
            smooth: true,
            data: props.data.map(item => [item[0], item[2]]),
            itemStyle: { color: '#ee6666' }
        },
        {
            name: '恢复人群 (R)',
            type: 'line',
            smooth: true,
            data: props.data.map(item => [item[0], item[3]]),
            itemStyle: { color: '#91cc75' }
        }
    ]
}))

const handleGlobalOut = () => {
    emit('hover-data-change', null)
}
</script>

<style scoped>
.visualization-section {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    height: 100%;
}

.chart {
    height: 100%;
    width: 100%;
}
</style> 