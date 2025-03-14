<template>
    <div class="sir-simulation dashboard">
        <div class="dashboard-header">
            <h1>SIR 传染病模型模拟</h1>
        </div>

        <div class="dashboard-content">
            <!-- 使用参数面板组件 -->
            <ParameterPanel v-model:beta="parameters.beta" v-model:gamma="parameters.gamma"
                v-model:totalPopulation="parameters.totalPopulation"
                v-model:initialInfected="parameters.initialInfected" v-model:simulationDays="parameters.simulationDays"
                v-model:timeStep="parameters.timeStep" v-model:isSync="isSync" :isRunning="isRunning" @start="startSimulation"
                @reset="resetSimulation" />
            <div class="main-content">
                <SIRChart :data="simulationData" @hover-data-change="currentHoverData = $event" />
                <DataTable :data="simulationData" />
            </div>
            <div class="statistics">
                <h3>当前数据</h3>
                <div v-if="currentHoverData" class="statistics-content">
                    <div class="statistics-header">第 {{ currentHoverData.day }} 天</div>
                    <div class="statistics-item">
                        <div class="item-label">
                            <span class="color-dot s-color"></span>
                            易感人群 (S):
                        </div>
                        <div class="item-value">
                            {{ Math.round(currentHoverData.S) }}
                            <span class="percentage">
                                ({{ ((currentHoverData.S / (currentHoverData.S + currentHoverData.I + currentHoverData.R)) * 100).toFixed(1) }}%)
                            </span>
                        </div>
                    </div>
                    <div class="statistics-item">
                        <div class="item-label">
                            <span class="color-dot i-color"></span>
                            感染人群 (I):
                        </div>
                        <div class="item-value">
                            {{ Math.round(currentHoverData.I) }}
                            <span class="percentage">
                                ({{ ((currentHoverData.I / (currentHoverData.S + currentHoverData.I + currentHoverData.R)) * 100).toFixed(1) }}%)
                            </span>
                        </div>
                    </div>
                    <div class="statistics-item">
                        <div class="item-label">
                            <span class="color-dot r-color"></span>
                            恢复人群 (R):
                        </div>
                        <div class="item-value">
                            {{ Math.round(currentHoverData.R) }}
                            <span class="percentage">
                                ({{ ((currentHoverData.R / (currentHoverData.S + currentHoverData.I + currentHoverData.R)) * 100).toFixed(1) }}%)
                            </span>
                        </div>
                    </div>
                    <div class="statistics-item total">
                        <div class="item-label">总人数:</div>
                        <div class="item-value">
                            {{ Math.round(currentHoverData.S + currentHoverData.I + currentHoverData.R) }}
                        </div>
                    </div>
                </div>
                <div v-else class="statistics-placeholder">
                    <p>鼠标悬停在图表上查看详细数据</p>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import ParameterPanel from '../components/ParameterPanel.vue'
import DataTable from '../components/DataTable.vue'
import SIRChart from '../components/SIRChart.vue'

// 状态定义
const parameters = ref({
    beta: 0.3,
    gamma: 0.1,
    totalPopulation: 1000,
    initialInfected: 1,
    simulationDays: 100,
    timeStep: 1
})

const isRunning = ref(false)
const days = ref(0)
const currentS = ref(0)
const currentI = ref(0)
const currentR = ref(0)
const simulationData = ref([])
const currentHoverData = ref(null)
const isSync = ref(true)

// 方法
const startSimulation = () => {
    if (isRunning.value) return
    isRunning.value = true

    try {
        const { beta, gamma, totalPopulation, initialInfected, timeStep, simulationDays } = parameters.value

        simulationData.value = []
        currentS.value = totalPopulation - initialInfected
        currentI.value = initialInfected
        currentR.value = 0
        days.value = 0

        // 一次性计算所有天数的数据
        for (let i = 0; i <= simulationDays; i++) {
            simulationData.value.push([
                days.value,
                currentS.value,
                currentI.value,
                currentR.value
            ])

            if (i < simulationDays) {
                // 计算SIR模型的变化
                const dS = -beta * currentS.value * currentI.value / totalPopulation
                const dI = beta * currentS.value * currentI.value / totalPopulation - gamma * currentI.value
                const dR = gamma * currentI.value

                currentS.value = Math.max(0, currentS.value + dS * timeStep)
                currentI.value = Math.max(0, currentI.value + dI * timeStep)
                currentR.value = Math.max(0, currentR.value + dR * timeStep)
                days.value++
            }
        }
        isRunning.value = false
    } catch (error) {
        console.error('Simulation error:', error)
        isRunning.value = false
    }
}

const resetSimulation = () => {
    isRunning.value = false
    days.value = 0
    const { totalPopulation, initialInfected } = parameters.value
    currentS.value = totalPopulation - initialInfected
    currentI.value = initialInfected
    currentR.value = 0
    simulationData.value = []
}

// 监听参数变化，实时更新
watch([() => parameters.value.beta, () => parameters.value.gamma, 
    () => parameters.value.totalPopulation, () => parameters.value.initialInfected,
    () => parameters.value.simulationDays], () => {
        console.log(parameters.value)
    if (isSync.value && !isRunning.value) {
        startSimulation()
    }
}, { deep: true, immediate: true })

</script>

<style scoped>
/* 仪表盘基础样式 */
.dashboard {
    min-height: 100vh;
    height: 100vh;
    /* 固定高度 */
    background-color: #f0f2f5;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    /* 防止整体滚动 */
}

.dashboard-header {
    background-color: #fff;
    padding: 16px 24px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
    z-index: 1;
}

.dashboard-header h1 {
    margin: 0;
    font-size: 1.5rem;
    color: #1a1a1a;
    font-weight: 500;
}

.dashboard-content {
    flex: 1;
    display: grid;
    grid-template-columns: 300px 1fr 250px;
    gap: 24px;
    padding: 24px;
    height: calc(100vh - 70px);
    overflow: hidden;
}

/* 主要内容区样式优化 */
.main-content {
    display: grid;
    grid-template-rows: 1fr 1fr;
    /* 固定图表高度 */
    gap: 24px;
    min-height: 0;
    /* 重要：允许grid子项收缩 */
}

.statistics {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.statistics h3 {
    margin: 0 0 16px 0;
    color: #1a1a1a;
    font-size: 1.1rem;
    font-weight: 500;
}

.statistics-content {
    display: flex;
    flex-direction: column;
    gap: 12px;
}

.statistics-header {
    font-weight: bold;
    margin-bottom: 4px;
    color: #1a1a1a;
    font-size: 1rem;
}

.statistics-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 8px 0;
    border-bottom: 1px solid #f0f0f0;
}

.statistics-item:last-child {
    border-bottom: none;
}

.item-label {
    display: flex;
    align-items: center;
    gap: 8px;
    color: #666;
    font-size: 0.9rem;
}

.color-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
}

.s-color {
    background-color: #5470c6;
}

.i-color {
    background-color: #ee6666;
}

.r-color {
    background-color: #91cc75;
}

.item-value {
    font-weight: 500;
    color: #1a1a1a;
    font-size: 0.9rem;
}

.percentage {
    color: #666;
    font-size: 0.85rem;
    margin-left: 4px;
}

.total {
    margin-top: 0px;
    padding-top: 0px;
    border-top: 0px solid #e8e8e8;
}

.total .item-label {
    font-weight: 500;
    color: #1a1a1a;
}

.statistics-placeholder {
    color: #666;
    font-size: 0.9rem;
    text-align: center;
    padding: 20px 0;
}
</style>