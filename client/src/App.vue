<template>
  <div class="app">
    <div v-if="isSidebarOpen" class="sidebar-scrim" @click="isSidebarOpen = false"></div>

    <aside class="sidebar" :class="{ open: isSidebarOpen }">
      <div class="sidebar-brand">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="subtitle">{{ t('nav.subtitle') }}</span>
      </div>
      <nav class="sidebar-nav">
        <router-link to="/" :class="{ active: $route.path === '/' }">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <rect x="2" y="2" width="7" height="7" rx="1.2" stroke="currentColor" stroke-width="1.5"/>
            <rect x="11" y="2" width="7" height="7" rx="1.2" stroke="currentColor" stroke-width="1.5"/>
            <rect x="2" y="11" width="7" height="7" rx="1.2" stroke="currentColor" stroke-width="1.5"/>
            <rect x="11" y="11" width="7" height="7" rx="1.2" stroke="currentColor" stroke-width="1.5"/>
          </svg>
          {{ t('nav.overview') }}
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M3 6.5L10 3L17 6.5V13.5L10 17L3 13.5V6.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
            <path d="M3 6.5L10 10M10 10L17 6.5M10 10V17" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
          </svg>
          {{ t('nav.inventory') }}
        </router-link>
        <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M2 3H4L6.3 13H15L17 6H5.2" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            <circle cx="7.5" cy="16.5" r="1.2" stroke="currentColor" stroke-width="1.5"/>
            <circle cx="14" cy="16.5" r="1.2" stroke="currentColor" stroke-width="1.5"/>
          </svg>
          {{ t('nav.orders') }}
        </router-link>
        <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M10 2.5V17.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            <path d="M13.5 5.8C13.5 4.3 12 3.3 10 3.3C8 3.3 6.5 4.4 6.5 6.1C6.5 9 13.5 8 13.5 11.4C13.5 13.3 12 14.3 10 14.3C8 14.3 6.5 13.3 6.5 11.8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          {{ t('nav.finance') }}
        </router-link>
        <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M2 14.5L7 9L10.5 12L18 4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M12.5 4H18V9.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          {{ t('nav.demandForecast') }}
        </router-link>
        <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
            <path d="M3 17V10.5M9.5 17V3M16 17V12.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          Reports
        </router-link>
      </nav>
      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="content-column">
      <button class="sidebar-toggle" @click="isSidebarOpen = !isSidebarOpen" aria-label="Toggle navigation" :aria-expanded="isSidebarOpen">
        <svg width="22" height="22" viewBox="0 0 22 22" fill="none">
          <path d="M3 6H19M3 11H19M3 16H19" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
        </svg>
      </button>
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { useRouter } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const router = useRouter()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])
    // Mobile drawer state for the sidebar; desktop ignores this (sidebar is always visible via CSS)
    const isSidebarOpen = ref(false)

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    // Close the mobile drawer whenever the route changes
    const stopAfterEach = router.afterEach(() => {
      isSidebarOpen.value = false
    })

    // Close the mobile drawer on Escape
    const handleEscape = (event) => {
      if (event.key === 'Escape') {
        isSidebarOpen.value = false
      }
    }

    onMounted(() => {
      loadTasks()
      window.addEventListener('keydown', handleEscape)
    })

    onUnmounted(() => {
      window.removeEventListener('keydown', handleEscape)
      stopAfterEach()
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      isSidebarOpen
    }
  }
}
</script>

<style>
:root {
  --ink: #0f172a;
  --slate: #64748b;
  --slate-dark: #475569;
  --border: #e2e8f0;
  --surface: #ffffff;
  --surface-muted: #f8fafc;
  --surface-hover: #f1f5f9;
  --accent: #2563eb;
  --accent-soft: #eff6ff;
  --success: #059669;
  --warning: #ea580c;
  --danger: #dc2626;
  --radius-control: 6px;
  --radius-surface: 10px;
  --shadow-rest: 0 1px 3px rgba(0, 0, 0, .05);
  --shadow-hover: 0 4px 12px rgba(0, 0, 0, .06);
  --shadow-modal: 0 20px 50px rgba(0, 0, 0, .15);
  --shadow-dropdown: 0 10px 25px rgba(0, 0, 0, .1);
  --z-sidebar: 100;
  --z-filterbar: 90;
  --z-drawer-overlay: 150;
  --z-drawer: 160;
  --z-dropdown: 1000;
  --z-modal: 2000;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--surface-muted);
  color: #1e293b;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app {
  display: flex;
  flex-direction: row;
  min-height: 100vh;
}

/* Hamburger toggle - hidden on desktop, shown at top of content column on mobile */
.sidebar-toggle {
  display: none;
}

.sidebar-scrim {
  display: none;
}

.sidebar {
  width: 256px;
  flex-shrink: 0;
  position: sticky;
  top: 0;
  height: 100vh;
  background: var(--surface);
  border-right: 1px solid var(--border);
  z-index: var(--z-sidebar);
  display: flex;
  flex-direction: column;
}

.sidebar-brand {
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
  padding: 1.5rem 1.25rem;
  border-bottom: 1px solid var(--border);
}

.sidebar-brand h1 {
  font-size: 1.375rem;
  font-weight: 700;
  color: var(--ink);
  letter-spacing: -0.025em;
}

.subtitle {
  font-size: 0.813rem;
  color: var(--slate);
  font-weight: 400;
}

.sidebar-nav {
  display: flex;
  flex-direction: column;
  padding: 0.75rem;
  gap: 0.125rem;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem 0.875rem;
  border-radius: var(--radius-control);
  color: var(--slate);
  font-weight: 500;
  font-size: 0.938rem;
  text-decoration: none;
  transition: all 0.15s ease;
}

.sidebar-nav a svg {
  flex-shrink: 0;
}

.sidebar-nav a:hover {
  background: var(--surface-hover);
  color: var(--ink);
}

.sidebar-nav a.active {
  background: var(--accent-soft);
  color: var(--accent);
  font-weight: 600;
}

.sidebar-footer {
  margin-top: auto;
  padding: 1rem 0.75rem;
  border-top: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.content-column {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
  max-width: 1400px;
  width: 100%;
  margin: 0 auto;
  padding: 1.5rem 2rem;
}

@media (max-width: 899px) {
  .sidebar {
    position: fixed;
    top: 0;
    left: 0;
    height: 100vh;
    transform: translateX(-100%);
    transition: transform 0.25s ease;
    box-shadow: var(--shadow-modal);
    /* Must render above .sidebar-scrim (--z-drawer-overlay) so nav links inside the open drawer are clickable */
    z-index: var(--z-drawer);
  }

  .sidebar.open {
    transform: translateX(0);
  }

  .sidebar-toggle {
    display: flex;
    align-items: center;
    justify-content: center;
    position: sticky;
    top: 0;
    z-index: calc(var(--z-sidebar) + 1);
    background: var(--surface);
    border: none;
    border-bottom: 1px solid var(--border);
    padding: 0.75rem 1rem;
    width: 100%;
    cursor: pointer;
    color: var(--ink);
  }

  .sidebar-scrim {
    display: block;
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.4);
    z-index: var(--z-drawer-overlay);
  }

  .content-column {
    width: 100%;
  }
}

.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: var(--ink);
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--slate);
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: var(--surface);
  padding: 1.25rem;
  border-radius: var(--radius-surface);
  border: 1px solid var(--border);
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: var(--shadow-hover);
}

.stat-label {
  color: var(--slate);
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: var(--ink);
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: var(--warning);
}

.stat-card.success .stat-value {
  color: var(--success);
}

.stat-card.danger .stat-value {
  color: var(--danger);
}

.stat-card.info .stat-value {
  color: var(--accent);
}

.card {
  background: var(--surface);
  border-radius: var(--radius-surface);
  padding: 1.25rem;
  border: 1px solid var(--border);
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid var(--border);
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--ink);
  letter-spacing: -0.025em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--surface-muted);
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: var(--slate-dark);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--surface-muted);
}

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: var(--radius-control);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: 3rem;
  color: var(--slate);
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>
