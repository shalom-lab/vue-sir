<template>
    <div class="data-section">
        <div class="data-header">
            <h3>模拟数据</h3>
            <button class="download-btn" @click="handleDownload" :disabled="!data.length">
                <!-- <i class="download-icon"></i> -->
                下载CSV
            </button>
        </div>
        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th v-for="header in headers" :key="header">{{ header }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="[day, S, I, R] in data" :key="day">
                        <td>{{ day }}</td>
                        <td>{{ Math.round(S) }}</td>
                        <td>{{ Math.round(I) }}</td>
                        <td>{{ Math.round(R) }}</td>
                        <td>{{ Math.round(S + I + R) }}</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script setup>
const props = defineProps({
    data: {
        type: Array,
        default: () => []
    },
    headers: {
        type: Array,
        default: () => ['天数', '易感人群(S)', '感染人群(I)', '恢复人群(R)', '总人数']
    }
})

const handleDownload = () => {
    if (!props.data.length) return

    const csvContent = [
        props.headers.join(','),
        ...props.data.map(row => {
            const [day, S, I, R] = row
            const total = S + I + R
            return [day, Math.round(S), Math.round(I), Math.round(R), Math.round(total)].join(',')
        })
    ].join('\n')

    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
    const link = document.createElement('a')
    const url = URL.createObjectURL(blob)
    link.setAttribute('href', url)
    link.setAttribute('download', `sir_simulation_data_${new Date().getTime()}.csv`)
    link.style.visibility = 'hidden'
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
}
</script>

<style scoped>
.data-section {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    display: grid;
    grid-template-rows: auto 1fr;
    min-height: 0;
    overflow: hidden;
}

.data-header {
    padding: 16px 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #e9ecef;
    background-color: white;
    z-index: 2;
}

.table-container {
    overflow-y: auto;
    height: 100%;
    padding: 0;
    position: relative;
}

table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.9rem;
}

thead {
    position: sticky;
    top: 0;
    z-index: 1;
    background-color: #f8f9fa;
}

th {
    position: sticky;
    top: 0;
    background-color: #f8f9fa;
    z-index: 1;
    padding: 12px 20px;
    text-align: right;
    border-bottom: 2px solid #dee2e6;
    font-weight: 600;
}

td {
    padding: 12px 20px;
    text-align: right;
    border-bottom: 1px solid #e9ecef;
    color: #212529;
}

th:first-child,
td:first-child {
    width: 80px;
}

th:not(:first-child),
td:not(:first-child) {
    width: calc((100% - 80px) / 4);
}

tbody tr:hover {
    background-color: #f8f9fa;
}

.download-btn {
    padding: 8px 16px;
    background-color: #3182ce;
    color: white;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 8px;
    transition: background-color 0.2s;
}

.download-btn:hover:not(:disabled) {
    background-color: #2c5282;
}

.download-btn:disabled {
    opacity: 0.7;
    cursor: not-allowed;
}

/* 滚动条样式 */
.table-container::-webkit-scrollbar {
    width: 8px;
}

.table-container::-webkit-scrollbar-track {
    background: #f1f1f1;
    border-radius: 4px;
}

.table-container::-webkit-scrollbar-thumb {
    background: #888;
    border-radius: 4px;
}

.table-container::-webkit-scrollbar-thumb:hover {
    background: #555;
}
</style> 