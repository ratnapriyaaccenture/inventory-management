<template>
  <div class="app-shell">

    <!-- Sidebar -->
    <AppSidebar
      @show-profile-details="showProfileDetails = true"
      @show-tasks="showTasks = true"
    />

    <!-- Main content -->
    <div class="app-main">

      <!-- Luxury animated background scene -->
      <div class="bg-scene" aria-hidden="true">
        <!-- Gradient orbs -->
        <div class="bg-orb bg-orb--a"></div>
        <div class="bg-orb bg-orb--b"></div>
        <div class="bg-orb bg-orb--c"></div>
        <div class="bg-orb bg-orb--d"></div>

        <!-- Floating package icons -->
        <span class="bg-pkg bg-pkg--1">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
        </span>
        <span class="bg-pkg bg-pkg--2">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
        </span>
        <span class="bg-pkg bg-pkg--3">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
        </span>
        <span class="bg-pkg bg-pkg--4">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
        </span>
        <span class="bg-pkg bg-pkg--5">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
        </span>
        <span class="bg-pkg bg-pkg--6">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
        </span>
        <span class="bg-pkg bg-pkg--7">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></svg>
        </span>

        <!-- Shopping cart watermark -->
        <span class="bg-cart">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"><circle cx="9" cy="21" r="1"/><circle cx="20" cy="21" r="1"/><path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"/></svg>
        </span>
      </div>

      <!-- Sticky top bar: FilterBar + LanguageSwitcher -->
      <div class="app-topbar">
        <FilterBar />
        <LanguageSwitcher />
      </div>

      <!-- Page content -->
      <main class="app-content">
        <router-view />
      </main>
    </div>

    <!-- Modals (unchanged) -->
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
import { ref, onMounted, computed, onUnmounted } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import AppSidebar from './components/AppSidebar.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher,
    AppSidebar
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

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

    onMounted(loadTasks)

    // ── Faux 3D card tilt effect ─────────────────────────────────────────────
    // Applies to every .card, .kpi-card, .stat-card, .trend-card, .hero-card
    // across all pages via event delegation — no view files need changing.
    const F3D_SELECTOR = '.card, .kpi-card, .stat-card, .trend-card, .hero-card'

    const initF3DCard = (card) => {
      if (card._f3dInited) return
      card._f3dInited = true

      // Inject shine overlay
      const shine = document.createElement('div')
      shine.className = 'f3d-shine'
      card.appendChild(shine)

      // Mark key inner elements for depth-float layer
      card.querySelectorAll(
        '.kpi-value, .stat-value, .hero-count, .trend-count, ' +
        '.kpi-label, .stat-label, .card-title, .trend-label, ' +
        '.hero-status, .progress-bar-wrap'
      ).forEach(el => el.classList.add('f3d-depth'))

      // Per-card reset on mouse leave
      card.addEventListener('mouseleave', () => {
        card.style.transition =
          'transform 0.75s cubic-bezier(0.23,1,0.32,1), box-shadow 0.75s cubic-bezier(0.23,1,0.32,1)'
        card.style.transform = ''
        card.style.boxShadow = ''
        shine.style.opacity = '0'
        card.querySelectorAll('.f3d-depth').forEach(el => {
          el.style.transform = 'translateZ(0px)'
        })
      })
    }

    const onF3DMove = (e) => {
      // Guard: document/window don't have .closest; only Element nodes do
      if (!e.target || typeof e.target.closest !== 'function') return
      const card = e.target.closest(F3D_SELECTOR)
      if (!card) return

      // Snap entrance animation to end so inline transform takes over cleanly
      if (card.getAnimations) {
        card.getAnimations().forEach(a => {
          if (a.playState === 'running') a.finish()
        })
      }

      initF3DCard(card)

      const rect  = card.getBoundingClientRect()
      const x     = e.clientX - rect.left
      const y     = e.clientY - rect.top
      const cx    = rect.width  / 2
      const cy    = rect.height / 2
      const nx    = (x - cx) / cx   // –1 … +1 horizontal
      const ny    = (y - cy) / cy   // –1 … +1 vertical

      const rotY  =  nx * 15        // tilt left ↔ right  (max ±15°)
      const rotX  = -ny * 11        // tilt up   ↕ down   (max ±11°)

      // Shadow moves opposite to tilt — sells depth
      const shX   = -rotY * 1.8
      const shY   =  rotX * 1.8 + 18

      card.style.transition =
        'transform 0.08s ease-out, box-shadow 0.08s ease-out'
      card.style.transform =
        `perspective(900px) rotateX(${rotX}deg) rotateY(${rotY}deg) scale3d(1.045,1.045,1.045)`
      card.style.boxShadow =
        `${shX}px ${shY}px 44px rgba(0,0,0,0.20),
         ${shX * 0.35}px ${shY * 0.35}px 16px rgba(0,0,0,0.12)`

      // Specular shine follows cursor
      const shine = card.querySelector('.f3d-shine')
      if (shine) {
        shine.style.opacity = '1'
        shine.style.background =
          `radial-gradient(circle at ${x}px ${y}px,
            rgba(255,255,255,0.48) 0%,
            rgba(255,255,255,0.14) 38%,
            transparent 65%)`
      }

      // Depth-float inner elements proportional to tilt magnitude
      const depth = Math.min(Math.sqrt(rotX * rotX + rotY * rotY) * 1.5, 26)
      card.querySelectorAll('.f3d-depth').forEach(el => {
        el.style.transform    = `translateZ(${depth}px)`
        el.style.transition   = 'transform 0.08s ease-out'
      })
    }

    onMounted(() => {
      document.addEventListener('mousemove', onF3DMove, { passive: true })
    })

    onUnmounted(() => {
      document.removeEventListener('mousemove', onF3DMove)
    })
    // ── End Faux 3D ──────────────────────────────────────────────────────────

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* ── Faux 3D Global Styles ───────────────────────────────────────────────── */

/* Enable 3D context on every card type */
.card,
.kpi-card,
.stat-card,
.trend-card,
.hero-card {
  transform-style: preserve-3d;
  will-change: transform;
  position: relative;   /* required so .f3d-shine can be absolute inside */
}

/* Specular shine overlay — injected at runtime per card */
.f3d-shine {
  position: absolute;
  inset: 0;
  border-radius: inherit;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 20;
  mix-blend-mode: overlay;
}

/* Depth-float layer for key inner elements */
.f3d-depth {
  transform-style: preserve-3d;
  will-change: transform;
}

/* Disable for users who prefer reduced motion */
@media (prefers-reduced-motion: reduce) {
  .card,
  .kpi-card,
  .stat-card,
  .trend-card,
  .hero-card {
    will-change: auto;
    transform: none !important;
  }
  .f3d-shine { display: none; }
  .f3d-depth { transform: none !important; }
}
/* ── End Faux 3D ─────────────────────────────────────────────────────────── */

:root {
  --sidebar-width: 240px;
  --sidebar-bg: #0f172a;
  --topbar-height: 56px;
  --content-bg: #f1f5f9;
  --card-bg: #ffffff;
  --card-border: #e2e8f0;
  --card-radius: 10px;
  --card-shadow: 0 1px 3px 0 rgba(0,0,0,0.08), 0 1px 2px -1px rgba(0,0,0,0.06);
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #94a3b8;
  --accent: #3b82f6;
  --accent-hover: #2563eb;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: var(--content-bg);
  color: var(--text-primary);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ── Shell layout ───────────────────────────────── */

.app-shell {
  display: flex;
  min-height: 100vh;
}

.app-main {
  flex: 1;
  margin-left: var(--sidebar-width);
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  min-width: 0;
}

.app-topbar {
  height: var(--topbar-height);
  background: var(--card-bg);
  border-bottom: 1px solid var(--card-border);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 1.5rem;
  position: sticky;
  top: 0;
  z-index: 50;
  gap: 1rem;
  flex-shrink: 0;
  /* Sit above the bg-scene */
  position: relative;
  z-index: 1;
}

.app-topbar > .filter-bar {
  flex: 1;
}

.app-content {
  flex: 1;
  padding: 1.5rem;
  max-width: 1400px;
  width: 100%;
  /* Sit above the bg-scene */
  position: relative;
  z-index: 1;
}

/* ── Page header ─────────────────────────────────── */

.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--text-muted);
  font-size: 0.875rem;
  margin-top: 3px;
}

/* ── Cards ───────────────────────────────────────── */

.card {
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: var(--card-radius);
  box-shadow: var(--card-shadow);
  margin-bottom: 1.25rem;
  overflow: hidden;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.25rem;
  border-bottom: 1px solid var(--card-border);
  gap: 1rem;
}

.card-title {
  font-size: 1rem;
  font-weight: 600;
  color: var(--text-primary);
}

/* ── Stat cards ──────────────────────────────────── */

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.stat-card {
  background: var(--card-bg);
  padding: 1.25rem;
  border-radius: var(--card-radius);
  border: 1px solid var(--card-border);
  box-shadow: var(--card-shadow);
  transition: box-shadow 0.2s ease;
}

.stat-card:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}

.stat-label {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--text-muted);
  margin-bottom: 0.5rem;
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.025em;
  line-height: 1;
}

.stat-card.success .stat-value { color: #059669; }
.stat-card.warning .stat-value { color: #d97706; }
.stat-card.danger  .stat-value { color: #dc2626; }
.stat-card.info    .stat-value { color: #2563eb; }

/* ── Tables ──────────────────────────────────────── */

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-bottom: 1px solid var(--card-border);
}

th {
  text-align: left;
  padding: 10px 12px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--text-secondary);
}

td {
  padding: 12px;
  border-bottom: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
  vertical-align: middle;
}

tbody tr:last-child td {
  border-bottom: none;
}

tbody tr {
  transition: background 0.12s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

/* ── Badges ──────────────────────────────────────── */

.badge {
  display: inline-block;
  padding: 3px 10px;
  border-radius: 5px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}

.badge.success    { background: #d1fae5; color: #065f46; }
.badge.warning    { background: #fef3c7; color: #92400e; }
.badge.danger     { background: #fee2e2; color: #991b1b; }
.badge.info       { background: #dbeafe; color: #1e40af; }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #fee2e2; color: #991b1b; }
.badge.stable     { background: #e0e7ff; color: #3730a3; }
.badge.high       { background: #fee2e2; color: #991b1b; }
.badge.medium     { background: #fef3c7; color: #92400e; }
.badge.low        { background: #dbeafe; color: #1e40af; }

/* ── Utility ─────────────────────────────────────── */

.loading {
  text-align: center;
  padding: 3rem;
  color: var(--text-muted);
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem 1.25rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.875rem;
}

.no-data-message {
  padding: 2rem;
  text-align: center;
  color: var(--text-muted);
  font-size: 0.875rem;
}

/* ── Background scene ────────────────────────────── */

.bg-scene {
  position: fixed;
  left: var(--sidebar-width);
  right: 0;
  top: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 0;
  overflow: hidden;
}

/* Scene base: luxury gradient wash across the whole content area */
.bg-scene {
  background:
    radial-gradient(ellipse 80% 50% at 0% 0%,   rgba(124,58,237,0.13) 0%, transparent 60%),
    radial-gradient(ellipse 60% 60% at 100% 100%, rgba(245,158,11,0.11) 0%, transparent 60%),
    radial-gradient(ellipse 55% 45% at 50% 55%,  rgba(29,78,216,0.08) 0%, transparent 60%);
}

/* Gradient orbs */
.bg-orb {
  position: absolute;
  border-radius: 50%;
  filter: blur(60px);
}

.bg-orb--a {
  top: -80px;
  left: -80px;
  width: 700px;
  height: 700px;
  background: radial-gradient(circle, rgba(124,58,237,0.28) 0%, rgba(139,92,246,0.14) 40%, transparent 70%);
  animation: orbDrift 18s ease-in-out infinite alternate;
}

.bg-orb--b {
  bottom: 60px;
  right: 40px;
  width: 580px;
  height: 580px;
  background: radial-gradient(circle, rgba(245,158,11,0.24) 0%, rgba(217,119,6,0.12) 40%, transparent 70%);
  animation: orbDrift 22s ease-in-out infinite alternate;
}

.bg-orb--c {
  top: 30%;
  left: 35%;
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, rgba(29,78,216,0.20) 0%, rgba(37,99,235,0.10) 40%, transparent 70%);
  animation: orbDrift 15s ease-in-out infinite alternate;
}

/* 4th orb: rose/magenta luxury accent */
.bg-orb--d {
  top: 55%;
  left: 5%;
  width: 420px;
  height: 420px;
  background: radial-gradient(circle, rgba(236,72,153,0.16) 0%, rgba(219,39,119,0.07) 40%, transparent 70%);
  animation: orbDrift 20s ease-in-out 3s infinite alternate;
}

@keyframes orbDrift {
  0%   { transform: translate(0, 0) scale(1); }
  33%  { transform: translate(30px, -20px) scale(1.05); }
  66%  { transform: translate(-20px, 28px) scale(0.96); }
  100% { transform: translate(25px, 15px) scale(1.03); }
}

/* Floating package icons */
.bg-pkg {
  position: absolute;
  display: block;
  animation: floatToCart var(--dur) ease-in-out var(--delay) infinite;
}

.bg-pkg svg {
  display: block;
  width: 100%;
  height: 100%;
  filter: drop-shadow(0 2px 6px var(--glow, rgba(139,92,246,0.4)));
}

@keyframes floatToCart {
  0%   { opacity: 0;    transform: translate(0, 0) scale(1) rotate(0deg); }
  8%   { opacity: 1; }
  85%  { opacity: 0.55; transform: translate(var(--tx), var(--ty)) scale(0.45) rotate(var(--rot)); }
  96%  { opacity: 0;    transform: translate(var(--tx), var(--ty)) scale(0.1) rotate(var(--rot)); }
  100% { opacity: 0;    transform: translate(0, 0) scale(1) rotate(0deg); }
}

/* pkg1 – violet */
.bg-pkg--1 {
  left: 5%;
  top: 14%;
  width: 36px;
  height: 36px;
  color: rgba(124,58,237,0.55);
  --glow: rgba(124,58,237,0.45);
  --tx: 900px; --ty: 580px; --rot: 30deg; --dur: 11s; --delay: 0s;
}

/* pkg2 – amber */
.bg-pkg--2 {
  left: 28%;
  top: 7%;
  width: 28px;
  height: 28px;
  color: rgba(245,158,11,0.55);
  --glow: rgba(245,158,11,0.40);
  --tx: 650px; --ty: 640px; --rot: -18deg; --dur: 9s; --delay: 2.5s;
}

/* pkg3 – blue */
.bg-pkg--3 {
  left: 60%;
  top: 9%;
  width: 32px;
  height: 32px;
  color: rgba(37,99,235,0.50);
  --glow: rgba(37,99,235,0.38);
  --tx: 320px; --ty: 620px; --rot: 45deg; --dur: 13s; --delay: 1s;
}

/* pkg4 – violet */
.bg-pkg--4 {
  left: 7%;
  top: 50%;
  width: 24px;
  height: 24px;
  color: rgba(124,58,237,0.50);
  --glow: rgba(124,58,237,0.38);
  --tx: 880px; --ty: 320px; --rot: -38deg; --dur: 8s; --delay: 4.5s;
}

/* pkg5 – gold */
.bg-pkg--5 {
  left: 20%;
  top: 74%;
  width: 30px;
  height: 30px;
  color: rgba(245,158,11,0.50);
  --glow: rgba(245,158,11,0.36);
  --tx: 760px; --ty: 115px; --rot: 22deg; --dur: 10s; --delay: 6s;
}

/* pkg6 – blue */
.bg-pkg--6 {
  left: 52%;
  top: 52%;
  width: 26px;
  height: 26px;
  color: rgba(37,99,235,0.48);
  --glow: rgba(37,99,235,0.34);
  --tx: 420px; --ty: 290px; --rot: -28deg; --dur: 7s; --delay: 3s;
}

/* pkg7 – rose */
.bg-pkg--7 {
  left: 76%;
  top: 28%;
  width: 34px;
  height: 34px;
  color: rgba(236,72,153,0.50);
  --glow: rgba(236,72,153,0.38);
  --tx: 170px; --ty: 500px; --rot: 55deg; --dur: 12s; --delay: 7.5s;
}

/* Shopping cart watermark */
.bg-cart {
  position: absolute;
  right: 3%;
  bottom: 5%;
  width: 260px;
  height: 260px;
  display: block;
  color: rgba(100,116,139,0.18);
  filter: drop-shadow(0 4px 24px rgba(124,58,237,0.18));
  animation: cartPulse 5s ease-in-out infinite;
}

.bg-cart svg {
  display: block;
  width: 100%;
  height: 100%;
}

@keyframes cartPulse {
  0%,100% { transform: scale(1);                opacity: 0.75; }
  50%      { transform: scale(1.07) rotate(2deg); opacity: 1; }
}

/* Respect reduced-motion preference */
@media (prefers-reduced-motion: reduce) {
  .bg-orb,
  .bg-pkg,
  .bg-cart {
    animation: none !important;
  }
}
</style>
