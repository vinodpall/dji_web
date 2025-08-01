<template>
  <div class="drone-control-main">
    <!-- 侧边栏菜单 -->
    <aside class="sidebar-menu">
      <div class="sidebar-tabs">
        <div
          v-for="tab in sidebarTabs"
          :key="tab.key"
          :class="['sidebar-tab', { active: route.path === tab.path }]"
          @click="handleTabClick(tab)"
        >
          <img :src="tab.icon" :alt="tab.label" />
        </div>
      </div>
    </aside>
    <!-- 主体内容区 -->
    <main class="main-content">
      <div class="main-flex">
        <section class="right-panel">
          <div class="mission-top-card card">
            <div class="mission-top-header">
              <img class="mission-top-logo" src="@/assets/source_data/bg_data/card_logo.png" alt="logo" />
              <span class="mission-top-title">任务记录</span>
            </div>
            <div class="mission-top-row">
              <span class="mission-lib-label">航线库</span>
              <div class="mission-top-selects">
                <div style="position: relative; display: inline-block;">
                  <select
                    v-model="selectedWaylineFile"
                    ref="recordTrackSelectRef"
                    class="mission-select treeselect-track"
                    style="width: 90px;"
                    @mousedown="onRecordTrackSelectMousedown"
                    @keydown="onRecordTrackSelectKeydown"
                    @blur="onRecordTrackSelectBlur"
                    @focus="onRecordTrackSelectFocus"
                    @change="onRecordTrackSelectChange"
                    :disabled="waylineFileLoading"
                  >
                    <option value="">全部</option>
                    <option v-for="file in waylineFiles" :key="file.wayline_id" :value="file.wayline_id">
                      {{ file.name }}
                    </option>
                  </select>
                  <span
                    class="custom-select-arrow"
                    @click="openRecordTrackSelect"
                    style="position: absolute; right: 10px; top: 50%; transform: translateY(-50%); width: 16px; height: 16px; display: flex; align-items: center; justify-content: center; cursor: pointer; z-index: 2;"
                  >
                    <svg width="12" height="12" viewBox="0 0 12 12">
                      <polygon v-if="!isRecordTrackSelectOpen" points="2,4 6,8 10,4" fill="#fff"/>
                      <polygon v-else points="2,8 6,4 10,8" fill="#fff"/>
                    </svg>
                  </span>
                </div>
                <button class="mission-btn mission-btn-pause">搜索</button>
              </div>
            </div>
          </div>
          <div class="mission-table-card card">
            <div class="mission-table-header">
              <div class="mission-th">序号</div>
              <div class="mission-th">任务名称</div>
              <div class="mission-th">航线名称</div>
              <div class="mission-th">任务类型</div>
              <div class="mission-th">状态</div>
              <div class="mission-th">开始时间</div>
              <div class="mission-th">结束时间</div>
              <div class="mission-th">创建人</div>
            </div>
            <div class="mission-table-body">
              <div v-if="loading" class="mission-loading">
                加载中...
              </div>
              <div v-else-if="error" class="mission-error">
                {{ error }}
              </div>
              <div v-else-if="jobs.length === 0" class="mission-empty">
                暂无任务记录
              </div>
              <div v-else class="mission-tr" v-for="(job, idx) in jobs" :key="job.job_id">
                <div class="mission-td">{{ idx + 1 }}</div>
                <div class="mission-td">{{ job.name }}</div>
                <div class="mission-td">{{ job.file_name || job.name }}</div>
                <div class="mission-td">{{ getTaskTypeText(job.task_type) }}</div>
                <div class="mission-td">
                  <span :class="['status-badge', getStatusClass(job.status)]">
                    {{ getStatusText(job.status) }}
                  </span>
                </div>
                <div class="mission-td">{{ formatTimestamp(job.begin_time) }}</div>
                <div class="mission-td">{{ formatTimestamp(job.completed_time) }}</div>
                <div class="mission-td">{{ job.username }}</div>
              </div>
            </div>
          </div>
        </section>
      </div>
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import trackListIcon from '@/assets/source_data/svg_data/track_list.svg'
import trackRecordsIcon from '@/assets/source_data/svg_data/track_records.svg'
import trackLogsIcon from '@/assets/source_data/svg_data/track_logs.svg'
import { useWaylineJobs, useDevices } from '../composables/useApi'

const router = useRouter()
const route = useRoute()

// 使用任务记录API
const { jobs, waylineFiles, loading, error, pagination, fetchJobs, fetchWaylineFiles, clearJobs } = useWaylineJobs()
const { getCachedWorkspaceId } = useDevices()

// 航线文件相关
const selectedWaylineFile = ref('')
const waylineFileLoading = ref(false)

// 加载航线文件列表
const loadWaylineFiles = async () => {
  const workspaceId = getCachedWorkspaceId()
  if (!workspaceId) {
    console.error('未找到workspace_id，无法加载航线文件')
    return
  }
  
  waylineFileLoading.value = true
  try {
    await fetchWaylineFiles(workspaceId, {
      page: 1,
      page_size: 100
    })
  } catch (err) {
    console.error('加载航线文件失败:', err)
  } finally {
    waylineFileLoading.value = false
  }
}

const sidebarTabs = [
  { key: 'list', label: '航线管理', icon: trackListIcon, path: '/dashboard/mission' },
  { key: 'records', label: '任务记录', icon: trackRecordsIcon, path: '/dashboard/mission-records' },
  { key: 'logs', label: '任务日志', icon: trackLogsIcon, path: '/dashboard/mission-logs' }
]

const handleTabClick = (tab: any) => {
  if (route.path !== tab.path) {
    router.push(tab.path)
  }
}

// 格式化时间戳
const formatTimestamp = (timestamp: string) => {
  if (!timestamp) return '-'
  const date = new Date(timestamp)
  return date.toLocaleString('zh-CN', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  })
}

// 获取任务状态文本
const getStatusText = (status: number) => {
  const statusMap: { [key: number]: string } = {
    [-1]: '未知状态',
    [0]: '待执行',
    [1]: '已发送',
    [2]: '执行中',
    [3]: '已暂停',
    [4]: '已取消',
    [5]: '执行成功',
    [6]: '执行失败'
  }
  return statusMap[status] || '未知'
}

// 获取任务状态样式
const getStatusClass = (status: number) => {
  const statusClassMap: { [key: number]: string } = {
    [-1]: 'status-unknown',
    [0]: 'status-pending',
    [1]: 'status-sent',
    [2]: 'status-running',
    [3]: 'status-paused',
    [4]: 'status-cancelled',
    [5]: 'status-success',
    [6]: 'status-failed'
  }
  return statusClassMap[status] || 'status-unknown'
}

// 获取任务类型文本
const getTaskTypeText = (taskType: number) => {
  const taskTypeMap: { [key: number]: string } = {
    [0]: '立即任务',
    [1]: '定时任务',
    [2]: '条件任务'
  }
  return taskTypeMap[taskType] || '未知'
}

// 加载任务记录数据
const loadJobRecords = async () => {
  const workspaceId = getCachedWorkspaceId()
  console.log('获取到的workspace_id:', workspaceId)
  console.log('localStorage中的workspace_id:', localStorage.getItem('workspace_id'))
  console.log('localStorage中的user:', localStorage.getItem('user'))
  console.log('当前选中的航线文件:', selectedWaylineFile.value)
  
  // 先清空现有数据
  clearJobs()
  
  if (!workspaceId) {
    console.error('未找到workspace_id，无法加载任务记录')
    // 尝试从用户信息中获取
    const userStr = localStorage.getItem('user')
    if (userStr) {
      try {
        const userData = JSON.parse(userStr)
        console.log('从localStorage解析的用户数据:', userData)
        if (userData.workspace_id) {
          console.log('从用户数据中找到workspace_id:', userData.workspace_id)
          await fetchJobs(userData.workspace_id, {
            page: 1,
            page_size: 10,
            file_id: selectedWaylineFile.value || undefined
          })
          return
        }
      } catch (err) {
        console.error('解析用户数据失败:', err)
      }
    }
    return
  }
  
  await fetchJobs(workspaceId, {
    page: 1,
    page_size: 10,
    file_id: selectedWaylineFile.value || undefined
  })
}

// 页面加载时获取数据
onMounted(async () => {
  await Promise.all([
    loadWaylineFiles(),
    loadJobRecords()
  ])
})

const recordTrackSelectRef = ref<HTMLSelectElement | null>(null)
const isRecordTrackSelectOpen = ref(false)
function openRecordTrackSelect() {
  if (recordTrackSelectRef.value) {
    recordTrackSelectRef.value.focus()
    recordTrackSelectRef.value.click && recordTrackSelectRef.value.click()
    isRecordTrackSelectOpen.value = true
  }
}
function onRecordTrackSelectBlur() {
  isRecordTrackSelectOpen.value = false
}
function onRecordTrackSelectFocus() {
  isRecordTrackSelectOpen.value = true
}
function onRecordTrackSelectChange() {
  isRecordTrackSelectOpen.value = false
  // 航线切换时清空当前数据并刷新任务列表
  console.log('航线切换，当前选中:', selectedWaylineFile.value)
  loadJobRecords()
}
function onRecordTrackSelectMousedown() {
  isRecordTrackSelectOpen.value = true
}
function onRecordTrackSelectKeydown(e: KeyboardEvent) {
  if ([" ", "Enter", "ArrowDown"].includes(e.key)) {
    isRecordTrackSelectOpen.value = true
  }
  if (["Escape", "Esc"].includes(e.key)) {
    isRecordTrackSelectOpen.value = false
  }
}
</script>

<style>
@import './mission-common.css';

/* 任务状态样式 */
.status-badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 500;
}

.status-pending {
  background: rgba(144, 147, 153, 0.2);
  color: #909399;
  border: 1px solid rgba(144, 147, 153, 0.3);
}

.status-sent {
  background: rgba(64, 158, 255, 0.2);
  color: #409eff;
  border: 1px solid rgba(64, 158, 255, 0.3);
}

.status-running {
  background: rgba(64, 158, 255, 0.2);
  color: #409eff;
  border: 1px solid rgba(64, 158, 255, 0.3);
}

.status-paused {
  background: rgba(230, 162, 60, 0.2);
  color: #e6a23c;
  border: 1px solid rgba(230, 162, 60, 0.3);
}

.status-cancelled {
  background: rgba(144, 147, 153, 0.2);
  color: #909399;
  border: 1px solid rgba(144, 147, 153, 0.3);
}

.status-completed {
  background: rgba(103, 194, 58, 0.2);
  color: #67c23a;
  border: 1px solid rgba(103, 194, 58, 0.3);
}

.status-success {
  background: rgba(103, 194, 58, 0.2);
  color: #67c23a;
  border: 1px solid rgba(103, 194, 58, 0.3);
}

.status-failed {
  background: rgba(245, 108, 108, 0.2);
  color: #f56c6c;
  border: 1px solid rgba(245, 108, 108, 0.3);
}

.status-unknown {
  background: rgba(144, 147, 153, 0.2);
  color: #909399;
  border: 1px solid rgba(144, 147, 153, 0.3);
}

/* 加载和错误状态样式 */
.mission-loading,
.mission-error,
.mission-empty {
  text-align: center;
  padding: 40px 20px;
  color: #909399;
  font-size: 14px;
}

.mission-error {
  color: #f56c6c;
}
</style>