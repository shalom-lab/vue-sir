<template>
    <div class="sidebar">
        <div class="parameter-group">
            <h3 class="group-title">模型参数</h3>
            <div class="parameter-list">
                <div class="parameter">
                    <div class="parameter-label">
                        <label>传染率 (β):</label>
                        <span class="parameter-value">{{ beta }}</span>
                    </div>
                    <input type="range" v-model.number="beta" min="0" max="1" step="0.01">
                </div>
                <div class="parameter">
                    <div class="parameter-label">
                        <label>恢复率 (γ):</label>
                        <span class="parameter-value">{{ gamma }}</span>
                    </div>
                    <input type="range" v-model.number="gamma" min="0" max="1" step="0.01">
                </div>
                <div class="parameter">
                    <div class="parameter-label">
                        <label>平均病程 (1/γ):</label>
                        <span class="parameter-value">{{ diseaseDuration.toFixed(1) }} 天</span>
                    </div>
                    <input type="range" :value="diseaseDuration" disabled min="1" max="100">
                </div>
                <div class="parameter">
                    <div class="parameter-label">
                        <label>基本传染数 (R₀):</label>
                        <span class="parameter-value">{{ basicReproductionNumber.toFixed(2) }}</span>
                    </div>
                    <input type="range" :value="basicReproductionNumber" disabled min="0" max="10" step="0.01">
                </div>
                <div class="parameter">
                    <div class="parameter-label">
                        <label>总人口:</label>
                        <span class="parameter-value">{{ totalPopulation }}</span>
                    </div>
                    <input type="range" v-model.number="totalPopulation" min="100" max="10000" step="100">
                </div>
                <div class="parameter">
                    <div class="parameter-label">
                        <label>初始感染者:</label>
                        <span class="parameter-value">{{ initialInfected }}</span>
                    </div>
                    <input type="range" v-model.number="initialInfected" min="1" :max="totalPopulation.value" step="1">
                </div>
                <div class="parameter">
                    <div class="parameter-label">
                        <label>时间步长:</label>
                        <span class="parameter-value">{{ timeStep }} 天</span>
                    </div>
                    <input type="range" v-model.number="timeStep" disabled step="1">
                </div>
                <div class="parameter">
                    <div class="parameter-label">
                        <label>模拟天数:</label>
                        <span class="parameter-value">{{ simulationDays }} 天</span>
                    </div>
                    <input type="range" v-model.number="simulationDays" min="1" max="365" step="1">
                </div>
            </div>
            <div class="parameter-summary">
                <div class="summary-list">
                    <div class="summary-item">
                        <span class="param-symbol">β =</span>
                        <span class="param-value">{{ beta.toFixed(2) }}</span>
                        <span class="param-desc">(传染率)</span>
                    </div>
                    <div class="summary-item">
                        <span class="param-symbol">γ =</span>
                        <span class="param-value">{{ gamma.toFixed(2) }}</span>
                        <span class="param-desc">(恢复率)</span>
                    </div>
                    <div class="summary-item">
                        <span class="param-symbol">R₀ =</span>
                        <span class="param-value">{{ basicReproductionNumber.toFixed(2) }}</span>
                        <span class="param-desc">(基本传染数)</span>
                    </div>
                    <div class="summary-item">
                        <span class="param-symbol">N =</span>
                        <span class="param-value">{{ totalPopulation }}</span>
                        <span class="param-desc">(总人口)</span>
                    </div>
                    <div class="summary-item">
                        <span class="param-symbol">I₀ =</span>
                        <span class="param-value">{{ initialInfected }}</span>
                        <span class="param-desc">(初始感染者)</span>
                    </div>
                </div>
            </div>
            <div class="sync-control">
                <label class="sync-label">
                    <input type="radio" v-model="isSync" :value="true" name="sync-mode">
                    <span>实时更新</span>
                </label>
                <label class="sync-label">
                    <input type="radio" v-model="isSync" :value="false" name="sync-mode">
                    <span>手动更新</span>
                </label>
            </div>
        </div>

        <div class="simulation-controls">
            <button class="primary-btn" @click="emit('start')" :disabled="isSync">
                {{ isRunning ? '模拟中...' : '开始模拟' }}
            </button>
            <button class="secondary-btn" @click="emit('reset')" :disabled="isSync">
                重置
            </button>
        </div>
    </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
    isRunning: Boolean
})

const emit = defineEmits(['start', 'reset'])

// 为每个参数定义单独的 model
const beta = defineModel('beta')
const gamma = defineModel('gamma')
const totalPopulation = defineModel('totalPopulation')
const initialInfected = defineModel('initialInfected')
const simulationDays = defineModel('simulationDays')
const timeStep = defineModel('timeStep')
const isSync = defineModel('isSync', { default: true })

// 计算属性
const basicReproductionNumber = computed(() => {
    return Number(beta.value) / Number(gamma.value)
})

const diseaseDuration = computed(() => {
    return 1 / Number(gamma.value)
})
</script>

<style scoped>
/* 左侧参数栏样式优化 */
.sidebar {
    background-color: #fff;
    border-radius: 8px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
    display: flex;
    flex-direction: column;
    padding: 0;
    overflow: hidden;
    height: 100%;
    /* 确保占满高度 */
}

.parameter-group {
    flex: 1;
    /* 让参数组占据剩余空间 */
    display: flex;
    flex-direction: column;
    overflow-y: auto;
    /* 允许滚动 */
}

.group-title {
    margin: 0;
    padding: 16px 20px;
    font-size: 1.1rem;
    font-weight: 500;
    color: #1a1a1a;
    background-color: #f8f9fa;
    border-bottom: 1px solid #e9ecef;
}

.parameter-list {
    padding: 16px 20px;
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.parameter {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.parameter-label {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.parameter-label label {
    font-size: 0.9rem;
    color: #4a5568;
    font-weight: 500;
}

.parameter-value {
    font-size: 0.9rem;
    color: #2d3748;
    font-weight: 600;
}

.parameter input[type="range"] {
    width: 100%;
    height: 6px;
    background: #e2e8f0;
    border-radius: 3px;
    cursor: pointer;
}

.parameter input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 16px;
    height: 16px;
    background: #3182ce;
    border-radius: 50%;
    cursor: pointer;
    transition: all 0.2s;
}

.parameter input[type="range"]::-webkit-slider-thumb:hover {
    transform: scale(1.1);
}

.parameter input[type="range"]:disabled {
    opacity: 0.7;
    cursor: not-allowed;
}

.sync-control {
    padding: 12px 20px;
    background-color: #f8f9fa;
    border-top: 1px solid #e9ecef;
    display: flex;
    gap: 24px;
    justify-content: center;
}

.sync-label {
    display: flex;
    align-items: center;
    gap: 6px;
    cursor: pointer;
    user-select: none;
    color: #4a5568;
    font-size: 0.9rem;
}

.sync-label input[type="radio"] {
    width: 16px;
    height: 16px;
    margin: 0;
    cursor: pointer;
    accent-color: #3182ce;
}

.sync-label:hover {
    color: #2d3748;
}

.parameter-summary {
    padding: 8px 0;
    background-color: #f8f9fa;
    border-top: none;
    font-size: 0.85rem;
    /* 减小字体大小 */
}

.summary-list {
    padding: 8px 16px;
}

.summary-item {
    display: flex;
    align-items: center;
    gap: 4px;
    /* 减小间距 */
    margin-bottom: 4px;
    /* 减小行间距 */
    font-family: "Courier New", monospace;
}

.param-symbol {
    font-weight: 500;
    color: #1a1a1a;
    min-width: 32px;
    /* 减小宽度 */
    text-align: right;
}

.param-value {
    font-weight: 600;
    color: #2c5282;
    min-width: 50px;
    /* 固定值的宽度 */
}

.param-desc {
    color: #666;
    font-family: system-ui, -apple-system, sans-serif;
}

.simulation-controls {
    padding: 12px 16px;
    display: flex;
    gap: 8px;
    border-top: 1px solid #e9ecef;
    background-color: white;
    /* 确保背景色 */
}

.primary-btn,
.secondary-btn {
    flex: 1;
    padding: 8px 12px;
    border: none;
    border-radius: 6px;
    font-size: 0.9rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s;
}

.primary-btn {
    background-color: #3182ce;
    color: white;
}

.primary-btn:hover:not(:disabled) {
    background-color: #2c5282;
}

.primary-btn:disabled,
.secondary-btn:disabled {
    opacity: 0.7;
    cursor: not-allowed;
}

.secondary-btn {
    background-color: #e2e8f0;
    color: #4a5568;
}

.secondary-btn:hover:not(:disabled) {
    background-color: #cbd5e0;
}
</style>