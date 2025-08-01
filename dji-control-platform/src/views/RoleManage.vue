<template>
  <div class="drone-control-main">
    <!-- 侧边栏 -->
    <aside class="sidebar-menu">
      <div class="sidebar-tabs">
        <div
          v-for="tab in sidebarTabs"
          :key="tab.key"
          :class="['sidebar-tab', { active: currentTab === tab.key }]"
          @click="handleTabClick(tab.key)"
        >
          <img :src="tab.icon" :alt="tab.label" />
        </div>
      </div>
    </aside>
    <!-- 主体内容区 -->
    <main class="main-content">
      <div class="main-flex">
        <section class="right-panel">
          <div class="mission-top-card card role-top-card">
            <div class="mission-top-header">
              <img class="mission-top-logo" src="@/assets/source_data/bg_data/card_logo.png" alt="logo" />
              <span class="mission-top-title">角色管理</span>
            </div>
            <div class="role-top-row">
              <label class="role-label">角色名称：</label>
              <input v-model="filter.roleName" class="role-input" placeholder="请输入角色名" />
              <button class="mission-btn mission-btn-pause" @click="onSearch">查询</button>
              <button class="mission-btn mission-btn-pause" @click="showAddRoleDialog = true">新增角色</button>
            </div>
          </div>
          <div class="mission-table-card card">
            <div class="mission-table-header">
              <div class="mission-th">序号</div>
              <div class="mission-th">角色名称</div>
              <div class="mission-th">角色介绍</div>
              <div class="mission-th">创建时间</div>
              <div class="mission-th">操作</div>
            </div>
            <div class="mission-table-body">
              <div class="mission-tr" v-for="(role, idx) in roles" :key="role.id">
                <div class="mission-td">{{ idx + 1 }}</div>
                <div class="mission-td">{{ role.role_name }}</div>
                <div class="mission-td">{{ role.role_description || '-' }}</div>
                <div class="mission-td">{{ formatTime(role.created_time) }}</div>
                <div class="mission-td">
                  <div class="role-action-btns">
                    <button class="icon-btn" title="查看" @click="openPermissionDialog(role)"><img :src="permissionIcon" /></button>
                    <button class="icon-btn" title="编辑" @click="openEditRoleDialog(role)"><img :src="editIcon" /></button>
                    <button class="icon-btn" title="删除" @click="openDeleteRoleDialog(role)"><img :src="deleteIcon" /></button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </section>
      </div>
    </main>
    <!-- 新增角色弹窗 -->
    <div v-if="showAddRoleDialog" class="custom-dialog-mask">
      <div class="custom-dialog">
        <div class="custom-dialog-title">新增角色</div>
        <div class="custom-dialog-content">
          <div class="add-role-form">
            <div class="add-role-form-row">
              <label>角色名称：</label>
              <input v-model="addRoleForm.roleName" class="role-input" placeholder="请输入角色名称" />
            </div>
            <div class="add-role-form-row">
              <label>角色介绍：</label>
              <input v-model="addRoleForm.roleDescription" class="role-input" placeholder="请输入角色介绍" />
            </div>
          </div>
        </div>
        <div class="custom-dialog-actions">
          <button class="mission-btn mission-btn-pause" @click="onAddRoleConfirm">确认</button>
          <button class="mission-btn mission-btn-cancel" @click="showAddRoleDialog = false">取消</button>
        </div>
      </div>
    </div>

    <!-- 权限管理弹窗 -->
    <div v-if="showPermissionDialog" class="custom-dialog-mask">
      <div class="permission-dialog">
        <div class="permission-dialog-header">
          <div class="permission-dialog-title">权限管理 - {{ currentRole?.role_name }}</div>
          <label class="select-all-checkbox">
            <input 
              type="checkbox" 
              v-model="selectAll"
              @change="handleSelectAll"
              class="permission-checkbox"
            />
            <span class="permission-label">全选</span>
          </label>
        </div>
        <div class="permission-dialog-content">
          <div class="permission-sections">
            <div v-for="section in permissionSections" :key="section.key" class="permission-section">
              <div class="permission-section-header">
                <div class="permission-section-title">{{ section.title }}</div>
                <label class="permission-item section-select-all">
                  <input 
                    type="checkbox" 
                    :checked="isSectionAllSelected(section)"
                    @change="toggleSectionAll(section)"
                    class="permission-checkbox"
                  />
                  <span class="permission-label">全选</span>
                </label>
              </div>
              <div class="permission-items">
                <label class="permission-item">
                  <input 
                    type="checkbox" 
                    v-model="selectedPermissions" 
                    :value="section.viewPermission"
                    class="permission-checkbox"
                  />
                  <span class="permission-label">页面查看</span>
                </label>
                <label v-for="permission in section.permissions" :key="permission.key" class="permission-item">
                  <input 
                    type="checkbox" 
                    v-model="selectedPermissions" 
                    :value="permission.key"
                    class="permission-checkbox"
                  />
                  <span class="permission-label">{{ permission.label }}</span>
                </label>
              </div>
            </div>
          </div>
        </div>
        <div class="permission-dialog-actions">
          <button class="mission-btn mission-btn-pause" @click="onPermissionConfirm">确认</button>
          <button class="mission-btn mission-btn-cancel" @click="showPermissionDialog = false">取消</button>
        </div>
      </div>
    </div>

    <!-- 编辑角色弹窗 -->
    <div v-if="showEditRoleDialog" class="custom-dialog-mask">
      <div class="custom-dialog">
        <div class="custom-dialog-title">编辑角色</div>
        <div class="custom-dialog-content">
          <div class="edit-role-form">
            <div class="edit-role-form-row">
              <label>角色名称：</label>
              <input v-model="editRoleForm.roleName" class="role-input" placeholder="请输入角色名称" />
            </div>
            <div class="edit-role-form-row">
              <label>角色介绍：</label>
              <input v-model="editRoleForm.roleDescription" class="role-input" placeholder="请输入角色介绍" />
            </div>
          </div>
        </div>
        <div class="custom-dialog-actions">
          <button class="mission-btn mission-btn-pause" @click="onEditRoleConfirm">确认</button>
          <button class="mission-btn mission-btn-cancel" @click="showEditRoleDialog = false">取消</button>
        </div>
      </div>
    </div>

    <!-- 删除角色确认弹窗 -->
    <div v-if="showDeleteRoleDialog" class="custom-dialog-mask">
      <div class="custom-dialog delete-confirm-dialog">
        <div class="custom-dialog-title">删除确认</div>
        <div class="custom-dialog-content">
          <div class="delete-confirm-message">
            <div class="delete-icon">⚠️</div>
            <div class="delete-text">
              确定要删除角色"{{ currentRole?.role_name }}"吗？删除后无法恢复，请谨慎操作。
            </div>
          </div>
        </div>
        <div class="custom-dialog-actions">
          <button class="mission-btn mission-btn-stop" @click="onDeleteRoleConfirm">确认删除</button>
          <button class="mission-btn mission-btn-cancel" @click="showDeleteRoleDialog = false">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useRoles } from '../composables/useApi'
import userIcon from '@/assets/source_data/svg_data/user.svg'
import roleIcon from '@/assets/source_data/svg_data/role.svg'
import permissionIcon from '@/assets/source_data/svg_data/permission.svg'
import editIcon from '@/assets/source_data/svg_data/edit.svg'
import deleteIcon from '@/assets/source_data/svg_data/delete.svg'

const router = useRouter()
const route = useRoute()

// 使用角色管理API
const { roles, loading, error, fetchRoles, createRole, updateRole, deleteRole } = useRoles()

const sidebarTabs = [
  { key: 'user', label: '用户管理', icon: userIcon, path: '/dashboard/users' },
  { key: 'role', label: '角色管理', icon: roleIcon, path: '/dashboard/roles' }
]
const currentTab = ref('role')
const handleTabClick = (key: string) => {
  const tab = sidebarTabs.find(t => t.key === key)
  if (tab && route.path !== tab.path) {
    router.push(tab.path)
  }
  currentTab.value = key
}

const filter = ref({
  roleName: ''
})
const onSearch = async () => {
  try {
    await fetchRoles({ 
      skip: 0, 
      limit: 100,
      search: filter.value.roleName 
    })
  } catch (err) {
    console.error('搜索角色失败:', err)
  }
}

const showAddRoleDialog = ref(false)
const showEditRoleDialog = ref(false)
const showDeleteRoleDialog = ref(false)
const addRoleForm = ref({
  roleName: '',
  roleDescription: ''
})
const editRoleForm = ref({
  roleName: '',
  roleDescription: ''
})

const onAddRoleConfirm = async () => {
  try {
    // 将表单数据转换为API需要的格式
    const apiRoleData = {
      role_name: addRoleForm.value.roleName,
      role_description: addRoleForm.value.roleDescription,
      permissions: []
    }
    
    await createRole(apiRoleData)
    showAddRoleDialog.value = false
    addRoleForm.value = { roleName: '', roleDescription: '' }
  } catch (err) {
    console.error('创建角色失败:', err)
  }
}

// 打开编辑角色弹窗
const openEditRoleDialog = (role: any) => {
  currentRole.value = role
  editRoleForm.value.roleName = role.role_name
  editRoleForm.value.roleDescription = role.role_description
  showEditRoleDialog.value = true
}

// 确认编辑角色
const onEditRoleConfirm = async () => {
  if (currentRole.value) {
    try {
      // 将表单数据转换为API需要的格式
      const apiRoleData = {
        role_name: editRoleForm.value.roleName,
        role_description: editRoleForm.value.roleDescription
      }
      
      await updateRole(currentRole.value.id.toString(), apiRoleData)
      showEditRoleDialog.value = false
      editRoleForm.value = { roleName: '', roleDescription: '' }
    } catch (err) {
      console.error('更新角色失败:', err)
    }
  }
}

// 打开删除角色确认弹窗
const openDeleteRoleDialog = (role: any) => {
  currentRole.value = role
  showDeleteRoleDialog.value = true
}

// 确认删除角色
const onDeleteRoleConfirm = async () => {
  if (currentRole.value) {
    try {
      await deleteRole(currentRole.value.id.toString())
      showDeleteRoleDialog.value = false
    } catch (err) {
      console.error('删除角色失败:', err)
    }
  }
}

// 格式化时间
const formatTime = (timeStr: string) => {
  if (!timeStr) return ''
  return new Date(timeStr).toLocaleString('zh-CN')
}

// 页面加载时获取角色列表
onMounted(async () => {
  try {
    await fetchRoles({ skip: 0, limit: 100 })
  } catch (err) {
    console.error('获取角色列表失败:', err)
  }
})

// 权限管理相关
const showPermissionDialog = ref(false)
const currentRole = ref<any>(null)
const selectedPermissions = ref<string[]>([])
const selectAll = ref(false)

// 权限配置
const permissionSections = ref([
  {
    key: 'home',
    title: '首页',
    viewPermission: 'home:view',
    permissions: [
      { key: 'home:task_dispatch', label: '下发任务' },
      { key: 'home:cancel_task', label: '取消任务' },
      { key: 'home:pause_route', label: '航线暂停' },
      { key: 'home:resume_route', label: '航线恢复' },
      { key: 'home:one_key_return', label: '一键返航' },
      { key: 'home:cancel_return', label: '取消返航' },
      { key: 'home:emergency_stop', label: '急停' }
    ]
  },
  {
    key: 'drone_control',
    title: '无人机控制',
    viewPermission: 'drone_control:view',
    permissions: [
      { key: 'drone_control:pause_route', label: '航线暂停' },
      { key: 'drone_control:stop_route', label: '航线停止' },
      { key: 'drone_control:remote_debug', label: '远程调试' },
      { key: 'drone_control:drone_control', label: '无人机控制' },
      { key: 'drone_control:gimbal_control', label: '云台控制' }
    ]
  },
  {
    key: 'dock_control',
    title: '机场控制',
    viewPermission: 'dock_control:view',
    permissions: [
      { key: 'dock_control:remote_debug', label: '远程调试' }
    ]
  },
  {
    key: 'route_manage',
    title: '航线管理',
    viewPermission: 'route_manage:view',
    permissions: [
      { key: 'route_manage:delete_folder', label: '删除文件夹' },
      { key: 'route_manage:delete_route', label: '删除航线' },
      { key: 'route_manage:add_route', label: '新增航线' },
      { key: 'route_manage:dispatch_task', label: '下发任务' },
      { key: 'route_manage:delete_waypoint', label: '删除航点' }
    ]
  },
  {
    key: 'task_records',
    title: '任务记录',
    viewPermission: 'task_records:view',
    permissions: []
  },
  {
    key: 'task_logs',
    title: '任务日志',
    viewPermission: 'task_logs:view',
    permissions: []
  },
  {
    key: 'device_manage',
    title: '设备管理',
    viewPermission: 'device_manage:view',
    permissions: [
      { key: 'device_manage:add_device', label: '添加设备' },
      { key: 'device_manage:delete_device', label: '删除设备' }
    ]
  },
  {
    key: 'alarm_log',
    title: '报警日志',
    viewPermission: 'alarm_log:view',
    permissions: []
  },
  {
    key: 'system_manage',
    title: '系统管理',
    viewPermission: 'system_manage:view',
    permissions: [
      { key: 'system_manage:add_user', label: '新增用户' },
      { key: 'system_manage:edit_user', label: '编辑用户' },
      { key: 'system_manage:delete_user', label: '删除用户' }
    ]
  },
  {
    key: 'role_manage',
    title: '角色管理',
    viewPermission: 'role_manage:view',
    permissions: [
      { key: 'role_manage:add_role', label: '新增角色' },
      { key: 'role_manage:permission_assign', label: '权限分配' },
      { key: 'role_manage:edit_role', label: '编辑角色' },
      { key: 'role_manage:delete_role', label: '删除角色' }
    ]
  }
])

// 打开权限管理弹窗
const openPermissionDialog = (role: any) => {
  currentRole.value = role
  selectedPermissions.value = role.permissions || []
  selectAll.value = false // Reset selectAll when opening dialog
  showPermissionDialog.value = true
}

// 处理全选/取消全选
const handleSelectAll = () => {
  if (selectAll.value) {
    // 包含页面查看权限和所有功能权限
    const allViewPermissions = permissionSections.value.map(section => section.viewPermission);
    const allFunctionPermissions = permissionSections.value.flatMap(section => 
      section.permissions.map(permission => permission.key)
    );
    selectedPermissions.value = [...allViewPermissions, ...allFunctionPermissions];
  } else {
    selectedPermissions.value = [];
  }
}

// 监听选中权限变化，更新全选状态
const updateSelectAllStatus = () => {
  const allViewPermissions = permissionSections.value.map(section => section.viewPermission);
  const allFunctionPermissions = permissionSections.value.flatMap(section => 
    section.permissions.map(permission => permission.key)
  );
  const allPermissions = [...allViewPermissions, ...allFunctionPermissions];
  selectAll.value = allPermissions.length > 0 && selectedPermissions.value.length === allPermissions.length;
}

// 监听selectedPermissions变化
watch(selectedPermissions, updateSelectAllStatus, { deep: true })

// 确认权限设置
const onPermissionConfirm = async () => {
  if (currentRole.value) {
    try {
      // 更新角色的权限
      const apiRoleData = {
        permissions: [...selectedPermissions.value]
      }
      
      await updateRole(currentRole.value.id.toString(), apiRoleData)
      showPermissionDialog.value = false
    } catch (err) {
      console.error('更新角色权限失败:', err)
    }
  }
}

// 选中当前页面所有权限
const selectSectionAll = (section: any) => {
  const sectionPermissions = [section.viewPermission, ...section.permissions.map((p: any) => p.key)];
  const allSectionSelected = sectionPermissions.every((perm: string) => selectedPermissions.value.includes(perm));
  
  if (allSectionSelected) {
    // 如果当前页面所有权限都已选中，则取消选中
    selectedPermissions.value = selectedPermissions.value.filter((perm: string) => !sectionPermissions.includes(perm));
  } else {
    // 如果当前页面权限未全部选中，则全选
    selectedPermissions.value = [...new Set([...selectedPermissions.value, ...sectionPermissions])];
  }
};

// 判断当前页面是否全选
const isSectionAllSelected = (section: any) => {
  const sectionPermissions = [section.viewPermission, ...section.permissions.map((p: any) => p.key)];
  return sectionPermissions.every((perm: string) => selectedPermissions.value.includes(perm));
};

// 切换当前页面全选状态
const toggleSectionAll = (section: any) => {
  const sectionPermissions = [section.viewPermission, ...section.permissions.map((p: any) => p.key)];
  if (isSectionAllSelected(section)) {
    selectedPermissions.value = selectedPermissions.value.filter((perm: string) => !sectionPermissions.includes(perm));
  } else {
    selectedPermissions.value = [...new Set([...selectedPermissions.value, ...sectionPermissions])];
  }
};
</script>

<style scoped>
@import './mission-common.css';
.role-top-card {
  min-height: 92px;
  padding-bottom: 10px;
}
.role-top-row {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-top: 8px;
  flex-wrap: wrap;
}
.role-label {
  color: #b6b6b6;
  font-size: 15px;
  margin-right: 2px;
}
.role-input {
  background: transparent;
  color: #fff;
  border: 1px solid #164159;
  border-radius: 4px;
  padding: 4px 12px;
  font-size: 14px;
  outline: none;
  min-width: 120px;
  margin-right: 0;
  height: 32px;
  line-height: 32px;
}
.role-action-btns {
  display: flex;
  gap: 8px;
  justify-content: center;
  align-items: center;
}
.icon-btn {
  background: transparent;
  border: none;
  padding: 0 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  transition: background 0.2s;
}
.icon-btn:hover {
  background: #223a5e44;
  border-radius: 4px;
}
.icon-btn img {
  width: 18px;
  height: 18px;
  object-fit: contain;
}
.add-role-form {
  display: flex;
  flex-direction: column;
  gap: 18px;
  min-width: 320px;
  max-width: 400px;
}
.add-role-form-row {
  display: flex;
  align-items: center;
  justify-content: flex-end;
}
.add-role-form label {
  color: #b6b6b6;
  font-size: 15px;
  width: 90px;
  text-align: right;
  margin-right: 18px;
  flex-shrink: 0;
}
.add-role-form .role-input {
  flex: 1;
  min-width: 0;
  max-width: 240px;
  width: 240px;
  margin-right: 0;
}
.edit-role-form {
  display: flex;
  flex-direction: column;
  gap: 18px;
  min-width: 320px;
  max-width: 400px;
}
.edit-role-form-row {
  display: flex;
  align-items: center;
  justify-content: flex-end;
}
.edit-role-form label {
  color: #b6b6b6;
  font-size: 15px;
  width: 90px;
  text-align: right;
  margin-right: 18px;
  flex-shrink: 0;
}
.edit-role-form .role-input {
  flex: 1;
  min-width: 0;
  max-width: 240px;
  width: 240px;
  margin-right: 0;
}
.delete-confirm-dialog {
  min-width: 380px;
  max-width: 420px;
}
.delete-confirm-message {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  text-align: left;
}
.delete-icon {
  font-size: 20px;
  flex-shrink: 0;
  margin-top: 2px;
}
.delete-text {
  color: #b6b6b6;
  font-size: 15px;
  line-height: 1.5;
  flex: 1;
}
.permission-dialog {
  background: linear-gradient(135deg, #1a233a 80%, #16213a 100%);
  border-radius: 18px;
  min-width: 800px;
  max-width: 1000px;
  max-height: 85vh;
  padding: 36px 44px 28px 44px;
  box-shadow: 0 8px 40px #000a, 0 2px 16px #59c0fc33;
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow: hidden;
}
.permission-dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  margin-bottom: 22px;
  padding-bottom: 8px;
  border-bottom: 1px solid #223a5e;
}
.permission-dialog-title {
  font-size: 22px;
  color: #67d5fd;
  font-weight: 700;
  letter-spacing: 1px;
}
.select-all-checkbox {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  color: #b6b6b6;
  font-size: 14px;
}
.select-all-checkbox .permission-checkbox {
  width: 16px;
  height: 16px;
  accent-color: #16bbf2;
  cursor: pointer;
}
.permission-dialog-content {
  width: 100%;
  margin-bottom: 32px;
  display: flex;
  flex-direction: column;
  align-items: center;
  max-height: 65vh;
  overflow-y: auto;
  padding-right: 8px;
}
.permission-dialog-content::-webkit-scrollbar {
  width: 6px;
  background: transparent;
}
.permission-dialog-content::-webkit-scrollbar-thumb {
  background: rgba(89, 192, 252, 0.3);
  border-radius: 3px;
  border: none;
}
.permission-dialog-content::-webkit-scrollbar-track {
  background: transparent;
}
.permission-dialog-content::-webkit-scrollbar-button {
  display: none;
  height: 0;
  width: 0;
}
.permission-dialog-content {
  scrollbar-width: thin;
  scrollbar-color: rgba(89, 192, 252, 0.3) transparent;
}
.permission-sections {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 24px;
}
.permission-section {
  border: 1px solid #223a5e;
  border-radius: 8px;
  padding: 16px;
  background: rgba(34, 58, 94, 0.2);
}
.permission-section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid #223a5e;
}
.permission-section-title {
  font-size: 16px;
  color: #67d5fd;
  font-weight: 600;
}
.permission-items {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 12px;
}
.permission-item {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  padding: 4px 0;
  transition: color 0.2s;
}
.permission-item:hover {
  color: #67d5fd;
}
.permission-checkbox {
  width: 16px;
  height: 16px;
  accent-color: #16bbf2;
  cursor: pointer;
}
.permission-label {
  font-size: 14px;
  color: #fff;
  cursor: pointer;
}
.section-select-all {
  margin-left: auto; /* Push to the right */
}
.permission-dialog-actions {
  display: flex;
  gap: 32px;
  justify-content: center;
  width: 100%;
  margin-top: 8px;
}
</style>