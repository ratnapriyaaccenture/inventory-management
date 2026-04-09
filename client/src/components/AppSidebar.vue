<template>
  <aside class="app-sidebar">
    <!-- Brand -->
    <div class="sidebar-brand">
      <div class="brand-mark">&#x2B21;</div>
      <div class="brand-text">
        <span class="brand-name">{{ t('nav.companyName') }}</span>
        <span class="brand-sub">{{ t('nav.subtitle') }}</span>
      </div>
    </div>

    <!-- Nav links -->
    <nav class="sidebar-nav">
      <router-link
        v-for="link in navLinks"
        :key="link.path"
        :to="link.path"
        :class="['nav-item', { active: isActive(link.path) }]"
      >
        <span class="nav-icon" aria-hidden="true">{{ link.icon }}</span>
        <span class="nav-label">{{ link.label || t(link.labelKey) }}</span>
      </router-link>
    </nav>

    <!-- User section at bottom -->
    <div class="sidebar-footer">
      <ProfileMenu
        @show-profile-details="$emit('show-profile-details')"
        @show-tasks="$emit('show-tasks')"
      />
    </div>
  </aside>
</template>

<script>
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'
import ProfileMenu from './ProfileMenu.vue'

export default {
  name: 'AppSidebar',
  components: { ProfileMenu },
  emits: ['show-profile-details', 'show-tasks'],
  setup() {
    const { t } = useI18n()
    const route = useRoute()

    const navLinks = [
      { path: '/',           labelKey: 'nav.overview',       icon: '▦' },
      { path: '/inventory',  labelKey: 'nav.inventory',      icon: '◫' },
      { path: '/orders',     labelKey: 'nav.orders',         icon: '◳' },
      { path: '/spending',   labelKey: 'nav.finance',        icon: '◈' },
      { path: '/demand',     labelKey: 'nav.demandForecast', icon: '◭' },
      { path: '/restocking', labelKey: 'nav.restocking',     icon: '⊞' },
      { path: '/reports',    label: 'Reports',               icon: '▤' },
    ]

    const isActive = (path) => {
      if (path === '/') return route.path === '/'
      return route.path === path
    }

    return { t, navLinks, isActive }
  }
}
</script>

<style scoped>
.app-sidebar {
  width: 240px;
  min-width: 240px;
  height: 100vh;
  background: #0f172a;
  display: flex;
  flex-direction: column;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 100;
  overflow-y: auto;
  overflow-x: hidden;
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 20px 16px 18px;
  border-bottom: 1px solid #1e293b;
  flex-shrink: 0;
}

.brand-mark {
  font-size: 1.4rem;
  color: #3b82f6;
  line-height: 1;
  flex-shrink: 0;
}

.brand-text {
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.brand-name {
  font-size: 0.875rem;
  font-weight: 700;
  color: #f1f5f9;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  letter-spacing: -0.01em;
}

.brand-sub {
  font-size: 0.688rem;
  color: #475569;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-top: 1px;
}

.sidebar-nav {
  flex: 1;
  padding: 10px 8px;
  display: flex;
  flex-direction: column;
  gap: 1px;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 9px 10px;
  border-radius: 7px;
  text-decoration: none;
  color: #64748b;
  font-size: 0.875rem;
  font-weight: 500;
  transition: background 0.15s ease, color 0.15s ease;
  border-left: 3px solid transparent;
  white-space: nowrap;
}

.nav-item:hover {
  background: #1e293b;
  color: #e2e8f0;
}

.nav-item.active {
  background: #1e3a5f;
  color: #93c5fd;
  border-left-color: #3b82f6;
}

.nav-icon {
  font-size: 1rem;
  width: 20px;
  text-align: center;
  flex-shrink: 0;
  opacity: 0.85;
}

.nav-label {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
}

.sidebar-footer {
  padding: 12px 8px;
  border-top: 1px solid #1e293b;
  flex-shrink: 0;
}
</style>
