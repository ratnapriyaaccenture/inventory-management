<template>
  <div class="orders">
    <div class="page-header">
      <h2>{{ t('orders.title') }}</h2>
      <p>{{ t('orders.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>

      <!-- ── Animated Hero Status Cards ───────────────────────────── -->
      <div class="status-hero-grid">
        <div
          v-for="(card, index) in statusCards"
          :key="card.status"
          :class="['hero-card', `hero-card--${card.colorKey}`, { 'hero-card--active': activeFilter === card.status }]"
          :style="{ '--anim-delay': `${index * 90}ms` }"
          @click="toggleFilter(card.status)"
          role="button"
          tabindex="0"
          @keydown.enter.prevent="toggleFilter(card.status)"
          @keydown.space.prevent="toggleFilter(card.status)"
          :aria-pressed="activeFilter === card.status"
        >
          <!-- Shimmer sweep (triggered on hover via CSS) -->
          <span class="hc-shimmer" aria-hidden="true"></span>

          <!-- Decorative floating circle in top-right -->
          <span class="hc-bg-circle" aria-hidden="true"></span>

          <!-- Top row: live pulse dot  +  icon box -->
          <div class="hc-top">
            <span class="hc-pulse" aria-hidden="true">
              <span class="hc-pulse__core"></span>
              <span class="hc-pulse__ring"></span>
            </span>
            <span class="hc-icon" v-html="card.icon" aria-hidden="true"></span>
          </div>

          <!-- Status label -->
          <p class="hc-label">{{ card.label }}</p>

          <!-- Animated count number (JS counter) -->
          <div class="hc-count">{{ animatedCounts[card.status] }}</div>

          <!-- Progress bar + percentage -->
          <div class="hc-bar-row">
            <div class="hc-bar-track">
              <div
                class="hc-bar-fill"
                :style="{ width: progressWidths[card.status] + '%' }"
              ></div>
            </div>
            <span class="hc-pct">{{ card.pct }}%</span>
          </div>

          <!-- "Active filter" badge – pops in when card is selected -->
          <transition name="badge-pop">
            <div v-if="activeFilter === card.status" class="hc-active-tag">
              Active filter &nbsp;×
            </div>
          </transition>
        </div>
      </div>

      <!-- ── Orders Table ──────────────────────────────────────────── -->
      <div class="card">
        <div class="card-header">
          <transition name="title-fade" mode="out-in">
            <h3 class="card-title" :key="activeFilter ?? 'all'">
              {{ activeFilter
                  ? `${t('status.' + activeFilter.toLowerCase())} ${t('orders.title')}`
                  : t('orders.allOrders') }}
              ({{ displayedOrders.length }})
            </h3>
          </transition>
          <transition name="btn-fade">
            <button
              v-if="activeFilter"
              class="clear-filter-btn"
              @click="toggleFilter(null)"
            >
              Show all orders
            </button>
          </transition>
        </div>

        <!-- Table fades + slides when filter changes -->
        <transition name="table-swap" mode="out-in">
          <div class="table-container" :key="activeFilter ?? 'all'">
            <table class="orders-table">
              <thead>
                <tr>
                  <th class="col-order-number">{{ t('orders.table.orderNumber') }}</th>
                  <th class="col-customer">{{ t('orders.table.customer') }}</th>
                  <th class="col-items">{{ t('orders.table.items') }}</th>
                  <th class="col-status">{{ t('orders.table.status') }}</th>
                  <th class="col-date">{{ t('orders.table.orderDate') }}</th>
                  <th class="col-date">{{ t('orders.table.expectedDelivery') }}</th>
                  <th class="col-value">{{ t('orders.table.totalValue') }}</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="order in displayedOrders" :key="order.order_number">
                  <td class="col-order-number"><strong>{{ order.order_number }}</strong></td>
                  <td class="col-customer">{{ translateCustomerName(order.customer) }}</td>
                  <td class="col-items">
                    <details class="items-details">
                      <summary class="items-summary">
                        {{ t('orders.itemsCount', { count: order.items.length }) }}
                      </summary>
                      <div class="items-dropdown">
                        <div v-for="(item, idx) in order.items" :key="idx" class="item-entry">
                          <span class="item-name">{{ translateProductName(item.name) }}</span>
                          <span class="item-meta">{{ t('orders.quantity') }}: {{ item.quantity }} @ {{ currencySymbol }}{{ item.unit_price }}</span>
                        </div>
                      </div>
                    </details>
                  </td>
                  <td class="col-status">
                    <span :class="['badge', getOrderStatusClass(order.status)]">
                      {{ t(`status.${order.status.toLowerCase()}`) }}
                    </span>
                  </td>
                  <td class="col-date">{{ formatDate(order.order_date) }}</td>
                  <td class="col-date">{{ formatDate(order.expected_delivery) }}</td>
                  <td class="col-value"><strong>{{ currencySymbol }}{{ order.total_value.toLocaleString() }}</strong></td>
                </tr>
              </tbody>
            </table>
          </div>
        </transition>
      </div>

      <!-- ── Submitted Restocking Orders ──────────────────────────── -->
      <div class="card" style="margin-top: 1.5rem;">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.submittedOrders') }}</h3>
        </div>
        <div v-if="restockingLoading" class="loading">{{ t('common.loading') }}</div>
        <div v-else-if="restockingError" class="error">{{ restockingError }}</div>
        <div
          v-else-if="restockingOrders.length === 0"
          class="no-data-message"
          style="padding: 1.5rem; text-align: center; color: #64748b;"
        >
          {{ t('restocking.noSubmittedOrders') }}
        </div>
        <div v-else class="table-container">
          <table style="width: 100%;">
            <thead>
              <tr>
                <th>{{ t('restocking.ordersTable.orderId') }}</th>
                <th>{{ t('restocking.ordersTable.date') }}</th>
                <th>{{ t('restocking.ordersTable.itemCount') }}</th>
                <th>{{ t('restocking.ordersTable.totalCost') }}</th>
                <th>{{ t('restocking.ordersTable.expectedDelivery') }}</th>
                <th>{{ t('restocking.ordersTable.status') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="order in restockingOrders" :key="order.id">
                <td><strong>#RST-{{ order.id }}</strong></td>
                <td>{{ formatDate(order.created_date) }}</td>
                <td>{{ order.items.length }} {{ t('common.items') }}</td>
                <td><strong>{{ formatCurrency(order.total_cost, currentCurrency) }}</strong></td>
                <td>{{ formatDate(order.expected_delivery_date) }}</td>
                <td><span class="badge info">{{ order.status }}</span></td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
import { ref, reactive, computed, watch, onMounted, nextTick } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'
import { formatCurrency } from '../utils/currency'

// ── Inline SVG icons (stroke="currentColor" inherits from .hc-icon color) ──
const SVG_CHECK = `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg>`
const SVG_ARROW = `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><line x1="5" y1="12" x2="19" y2="12"/><polyline points="12 5 19 12 12 19"/></svg>`
const SVG_CLOCK = `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>`
const SVG_ALERT = `<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>`

// Static config — colorKey maps to .hero-card--{colorKey} CSS variant
const STATUS_META = [
  { status: 'Delivered',   colorKey: 'success', labelKey: 'status.delivered',   icon: SVG_CHECK },
  { status: 'Shipped',     colorKey: 'info',    labelKey: 'status.shipped',     icon: SVG_ARROW },
  { status: 'Processing',  colorKey: 'warning', labelKey: 'status.processing',  icon: SVG_CLOCK },
  { status: 'Backordered', colorKey: 'danger',  labelKey: 'status.backordered', icon: SVG_ALERT },
]

export default {
  name: 'Orders',
  setup() {
    const { t, currentCurrency, translateProductName, translateCustomerName } = useI18n()
    const currencySymbol = computed(() => currentCurrency.value === 'JPY' ? '¥' : '$')

    // ── Core state ───────────────────────────────────────────────
    const loading           = ref(true)
    const error             = ref(null)
    const orders            = ref([])
    const restockingOrders  = ref([])
    const restockingLoading = ref(false)
    const restockingError   = ref(null)

    // Which status card is selected as a filter (null = show all)
    const activeFilter = ref(null)

    // Animation state – reactive objects for O(1) key access
    const animatedCounts = reactive({ Delivered: 0, Shipped: 0, Processing: 0, Backordered: 0 })
    const progressWidths = reactive({ Delivered: 0, Shipped: 0, Processing: 0, Backordered: 0 })

    // ── Global filters ───────────────────────────────────────────
    const { selectedPeriod, selectedLocation, selectedCategory, selectedStatus, getCurrentFilters } = useFilters()

    // ── Computed ─────────────────────────────────────────────────
    const getByStatus = (status) => orders.value.filter(o => o.status === status)

    const statusCards = computed(() => {
      const total = orders.value.length
      return STATUS_META.map(m => ({
        ...m,
        label: t(m.labelKey),
        count: getByStatus(m.status).length,
        pct:   total > 0 ? Math.round(getByStatus(m.status).length / total * 100) : 0,
      }))
    })

    const displayedOrders = computed(() =>
      activeFilter.value
        ? orders.value.filter(o => o.status === activeFilter.value)
        : orders.value
    )

    // ── Animation helpers ────────────────────────────────────────
    // Smooth ease-out cubic for the number counter
    const easeOutCubic = (x) => 1 - Math.pow(1 - x, 3)

    const animateCount = (status, target) => {
      const DURATION = 1300
      const t0 = performance.now()
      const tick = (now) => {
        const p = Math.min((now - t0) / DURATION, 1)
        animatedCounts[status] = Math.round(easeOutCubic(p) * target)
        if (p < 1) requestAnimationFrame(tick)
      }
      requestAnimationFrame(tick)
    }

    const triggerAnimations = () => {
      const total = orders.value.length
      STATUS_META.forEach((m, i) => {
        const count = getByStatus(m.status).length
        // Stagger counter starts (matches card entrance stagger of 90ms)
        setTimeout(() => animateCount(m.status, count), i * 90)
        // Progress bars begin after card entrance animations finish (~500ms)
        setTimeout(() => {
          progressWidths[m.status] = total > 0 ? Math.round(count / total * 100) : 0
        }, 480 + i * 60)
      })
    }

    const resetAnimations = () => {
      STATUS_META.forEach(m => {
        animatedCounts[m.status] = 0
        progressWidths[m.status] = 0
      })
    }

    // ── Interaction ──────────────────────────────────────────────
    const toggleFilter = (status) => {
      // Click active card again → deselect; click null → show all
      activeFilter.value = (status && activeFilter.value !== status) ? status : null
    }

    // ── Data loading ─────────────────────────────────────────────
    const loadOrders = async () => {
      try {
        loading.value = true
        error.value = null
        const fetched = await api.getOrders(getCurrentFilters())
        orders.value = fetched.sort((a, b) => new Date(a.order_date) - new Date(b.order_date))
      } catch (err) {
        error.value = 'Failed to load orders: ' + err.message
      } finally {
        loading.value = false
      }
      // Wait for Vue to flush the DOM (v-else block now rendered) then animate
      await nextTick()
      if (orders.value.length > 0) triggerAnimations()
    }

    const loadRestockingOrders = async () => {
      try {
        restockingLoading.value = true
        restockingError.value = null
        restockingOrders.value = await api.getRestockingOrders()
      } catch (err) {
        restockingError.value = 'Failed to load restocking orders: ' + err.message
      } finally {
        restockingLoading.value = false
      }
    }

    watch([selectedPeriod, selectedLocation, selectedCategory, selectedStatus], () => {
      resetAnimations()
      activeFilter.value = null
      loadOrders()
    })

    const getOrdersByStatus  = (status) => getByStatus(status)
    const getOrderStatusClass = (status) => {
      const map = { Delivered: 'success', Shipped: 'info', Processing: 'warning', Backordered: 'danger' }
      return map[status] || 'info'
    }

    const formatDate = (dateString) => {
      const { currentLocale } = useI18n()
      const locale = currentLocale.value === 'ja' ? 'ja-JP' : 'en-US'
      return new Date(dateString).toLocaleDateString(locale, {
        year: 'numeric', month: 'short', day: 'numeric'
      })
    }

    onMounted(() => {
      loadOrders()
      loadRestockingOrders()
    })

    return {
      t, loading, error, orders,
      activeFilter, animatedCounts, progressWidths,
      statusCards, displayedOrders,
      toggleFilter, getOrdersByStatus, getOrderStatusClass, formatDate,
      currencySymbol, translateProductName, translateCustomerName,
      restockingOrders, restockingLoading, restockingError,
      currentCurrency, formatCurrency,
    }
  }
}
</script>

<style scoped>
/* ═══════════════════════════════════════════════════════════════
   HERO STATUS CARDS  — grid + card base
═══════════════════════════════════════════════════════════════ */

.status-hero-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
  margin-bottom: 1.5rem;
}

/* ── Base card ─────────────────────────────────────────────── */
.hero-card {
  position: relative;
  overflow: hidden;
  border-radius: 16px;
  padding: 1.25rem 1.375rem 1.125rem;
  cursor: pointer;
  user-select: none;
  border: 1.5px solid transparent;

  /* Staggered entrance: slide-up + spring pop */
  animation: heroCardIn 0.55s cubic-bezier(0.34, 1.56, 0.64, 1) var(--anim-delay, 0ms) both;

  /* Smooth hover/active transitions */
  transition:
    transform   0.28s cubic-bezier(0.34, 1.56, 0.64, 1),
    box-shadow  0.28s ease,
    border-color 0.2s ease;

  will-change: transform;
}

/* Hover: lift + glow */
.hero-card:hover {
  transform: translateY(-5px) scale(1.025);
  box-shadow: 0 16px 40px var(--card-glow), 0 4px 12px rgba(0, 0, 0, 0.06);
  border-color: var(--card-accent);
}

/* Press feedback */
.hero-card:active {
  transform: translateY(-1px) scale(0.975);
  transition: transform 0.08s ease;
}

/* Keyboard focus */
.hero-card:focus-visible {
  outline: 3px solid var(--card-accent);
  outline-offset: 3px;
}

/* ── Color variants (CSS custom properties per theme) ─────── */
.hero-card--success {
  background: linear-gradient(140deg, #f0fdf4 0%, #dcfce7 100%);
  border-color: rgba(22, 163, 74, 0.16);
  --card-accent:       #16a34a;
  --card-accent-muted: rgba(22, 163, 74, 0.13);
  --card-glow:         rgba(22, 163, 74, 0.26);
  --card-text:         #15803d;
  --card-bar:          #16a34a;
  --card-circle-bg:    rgba(22, 163, 74, 0.07);
}

.hero-card--info {
  background: linear-gradient(140deg, #eff6ff 0%, #dbeafe 100%);
  border-color: rgba(37, 99, 235, 0.16);
  --card-accent:       #2563eb;
  --card-accent-muted: rgba(37, 99, 235, 0.13);
  --card-glow:         rgba(37, 99, 235, 0.26);
  --card-text:         #1d4ed8;
  --card-bar:          #2563eb;
  --card-circle-bg:    rgba(37, 99, 235, 0.07);
}

.hero-card--warning {
  background: linear-gradient(140deg, #fffbeb 0%, #fef3c7 100%);
  border-color: rgba(217, 119, 6, 0.16);
  --card-accent:       #d97706;
  --card-accent-muted: rgba(217, 119, 6, 0.13);
  --card-glow:         rgba(217, 119, 6, 0.26);
  --card-text:         #b45309;
  --card-bar:          #d97706;
  --card-circle-bg:    rgba(217, 119, 6, 0.07);
}

.hero-card--danger {
  background: linear-gradient(140deg, #fef2f2 0%, #fee2e2 100%);
  border-color: rgba(220, 38, 38, 0.16);
  --card-accent:       #dc2626;
  --card-accent-muted: rgba(220, 38, 38, 0.13);
  --card-glow:         rgba(220, 38, 38, 0.26);
  --card-text:         #b91c1c;
  --card-bar:          #dc2626;
  --card-circle-bg:    rgba(220, 38, 38, 0.07);
}

/* ── Active / selected state ──────────────────────────────── */
.hero-card--active {
  border-color: var(--card-accent) !important;
  box-shadow:
    0 0 0 4px var(--card-accent-muted),
    0 12px 32px var(--card-glow) !important;
}

/* ── Shimmer sweep on hover ───────────────────────────────── */
.hc-shimmer {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    105deg,
    transparent 20%,
    rgba(255, 255, 255, 0.52) 50%,
    transparent 80%
  );
  transform: translateX(-100%) skewX(-15deg);
  pointer-events: none;
}

.hero-card:hover .hc-shimmer {
  animation: shimmerSweep 0.65s ease forwards;
}

/* ── Decorative floating background circle ────────────────── */
.hc-bg-circle {
  position: absolute;
  right: -28px;
  top: -28px;
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background: var(--card-circle-bg);
  pointer-events: none;
  animation: bgCircleFloat 5s ease-in-out infinite;
  animation-delay: var(--anim-delay, 0ms);
}

/* ── Top row ─────────────────────────────────────────────── */
.hc-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 1rem;
}

/* Live pulse dot */
.hc-pulse {
  position: relative;
  width: 12px;
  height: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.hc-pulse__core {
  position: absolute;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: var(--card-accent);
  animation: pulseCore 2.8s ease-in-out infinite;
  animation-delay: var(--anim-delay, 0ms);
}

.hc-pulse__ring {
  position: absolute;
  width: 9px;
  height: 9px;
  border-radius: 50%;
  border: 2px solid var(--card-accent);
  animation: pulseRing 2.8s ease-out infinite;
  animation-delay: var(--anim-delay, 0ms);
}

/* Icon box */
.hc-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  border-radius: 10px;
  background: var(--card-accent-muted);
  color: var(--card-accent);
  flex-shrink: 0;
  /* Subtle float cycle — offset per card using --anim-delay */
  animation: iconFloat 3.8s ease-in-out infinite;
  animation-delay: calc(var(--anim-delay, 0ms) + 400ms);
}

/* ── Label ───────────────────────────────────────────────── */
.hc-label {
  font-size: 0.68rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--card-text);
  margin: 0 0 0.35rem;
  opacity: 0.85;
}

/* ── Animated count (JS rAF counter fills this) ───────────── */
.hc-count {
  font-size: 2.8rem;
  font-weight: 800;
  line-height: 1;
  color: #0f172a;
  letter-spacing: -0.04em;
  margin-bottom: 0.9rem;
  font-variant-numeric: tabular-nums;
}

/* ── Progress bar row ────────────────────────────────────── */
.hc-bar-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.hc-bar-track {
  flex: 1;
  height: 5px;
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.09);
  overflow: hidden;
}

.hc-bar-fill {
  height: 100%;
  border-radius: 999px;
  background: var(--card-bar);
  width: 0%;
  /* Spring-easing transition — fires when JS sets progressWidths */
  transition: width 0.95s cubic-bezier(0.34, 1.4, 0.64, 1);
}

.hc-pct {
  font-size: 0.73rem;
  font-weight: 600;
  color: var(--card-text);
  min-width: 32px;
  text-align: right;
}

/* ── "Active filter" badge ───────────────────────────────── */
.hc-active-tag {
  position: absolute;
  bottom: 10px;
  right: 10px;
  font-size: 0.62rem;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  padding: 3px 9px;
  border-radius: 999px;
  background: var(--card-accent);
  color: #ffffff;
  pointer-events: none;
}

/* ═══════════════════════════════════════════════════════════════
   KEYFRAME ANIMATIONS
═══════════════════════════════════════════════════════════════ */

/* 1. Card entrance: slide up + spring overshoot */
@keyframes heroCardIn {
  0%   { opacity: 0; transform: translateY(28px) scale(0.91); }
  55%  { opacity: 1; }
  100% { opacity: 1; transform: translateY(0)    scale(1);    }
}

/* 2. Shimmer sweep across card on hover */
@keyframes shimmerSweep {
  from { transform: translateX(-100%) skewX(-15deg); }
  to   { transform: translateX(200%)  skewX(-15deg); }
}

/* 3. Background circle gentle float */
@keyframes bgCircleFloat {
  0%,  100% { transform: translate(0, 0)      scale(1);    }
  33%        { transform: translate(-7px, 7px) scale(1.06); }
  66%        { transform: translate(5px, -5px) scale(0.94); }
}

/* 4. Pulse core: breathe in/out */
@keyframes pulseCore {
  0%,  100% { transform: scale(1);    opacity: 1;    }
  50%        { transform: scale(0.82); opacity: 0.65; }
}

/* 5. Pulse ring: expand outward and fade */
@keyframes pulseRing {
  0%   { transform: scale(1);   opacity: 0.85; }
  75%  { transform: scale(3);   opacity: 0;    }
  100% { transform: scale(3);   opacity: 0;    }
}

/* 6. Icon box: gentle vertical float */
@keyframes iconFloat {
  0%,  100% { transform: translateY(0);    }
  50%        { transform: translateY(-4px); }
}

/* ═══════════════════════════════════════════════════════════════
   VUE TRANSITION CLASSES
═══════════════════════════════════════════════════════════════ */

/* Badge pop-in / pop-out */
.badge-pop-enter-active {
  animation: badgePopIn 0.32s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}
.badge-pop-leave-active {
  animation: badgePopIn 0.14s ease reverse forwards;
}
@keyframes badgePopIn {
  from { opacity: 0; transform: scale(0.55) translateY(4px); }
  to   { opacity: 1; transform: scale(1)    translateY(0);   }
}

/* Table fade-swap when filter changes (uses :key trick) */
.table-swap-enter-active { transition: opacity 0.22s ease, transform 0.22s ease; }
.table-swap-leave-active { transition: opacity 0.14s ease, transform 0.14s ease; }
.table-swap-enter-from   { opacity: 0; transform: translateY(10px); }
.table-swap-leave-to     { opacity: 0; transform: translateY(-6px); }

/* Card title cross-fade */
.title-fade-enter-active { transition: opacity 0.16s ease, transform 0.16s ease; }
.title-fade-leave-active { transition: opacity 0.1s  ease, transform 0.1s  ease; }
.title-fade-enter-from   { opacity: 0; transform: translateY(6px);  }
.title-fade-leave-to     { opacity: 0; transform: translateY(-4px); }

/* "Show all orders" button slide in from right */
.btn-fade-enter-active { transition: opacity 0.2s ease, transform 0.2s ease; }
.btn-fade-leave-active { transition: opacity 0.14s ease;                      }
.btn-fade-enter-from   { opacity: 0; transform: translateX(10px); }
.btn-fade-leave-to     { opacity: 0; }

/* ═══════════════════════════════════════════════════════════════
   CLEAR FILTER BUTTON
═══════════════════════════════════════════════════════════════ */

.clear-filter-btn {
  font-size: 0.8125rem;
  font-weight: 600;
  padding: 0.375rem 0.875rem;
  border-radius: 8px;
  border: 1.5px solid #e2e8f0;
  background: #ffffff;
  color: #475569;
  cursor: pointer;
  white-space: nowrap;
  transition: background 0.15s ease, border-color 0.15s ease, color 0.15s ease;
}

.clear-filter-btn:hover {
  background: #f8fafc;
  border-color: #cbd5e1;
  color: #0f172a;
}

/* ═══════════════════════════════════════════════════════════════
   TABLE STYLES (unchanged from original)
═══════════════════════════════════════════════════════════════ */

.orders-table {
  table-layout: fixed;
  width: 100%;
}

.col-order-number { width: 130px; }
.col-customer     { width: 180px; }
.col-items        { width: 200px; }
.col-status       { width: 130px; }
.col-date         { width: 140px; }
.col-value        { width: 120px; }

.items-details { position: relative; }

.items-summary {
  cursor: pointer;
  color: #3b82f6;
  font-weight: 500;
  list-style: none;
  user-select: none;
  display: inline-block;
}
.items-summary::-webkit-details-marker { display: none; }
.items-summary::before {
  content: '▶';
  display: inline-block;
  margin-right: 0.375rem;
  font-size: 0.75rem;
  transition: transform 0.2s;
}
.items-details[open] .items-summary::before { transform: rotate(90deg); }
.items-summary:hover { color: #2563eb; text-decoration: underline; }

.items-dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  margin-top: 0.5rem;
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -1px rgba(0,0,0,0.06);
  padding: 0.75rem;
  z-index: 10;
  min-width: 300px;
  max-width: 400px;
}

.item-entry {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  padding: 0.5rem;
  border-bottom: 1px solid #f1f5f9;
}
.item-entry:last-child { border-bottom: none; }
.item-name { font-size: 0.875rem; font-weight: 500; color: #0f172a; }
.item-meta { font-size: 0.813rem; color: #64748b; }

/* ═══════════════════════════════════════════════════════════════
   RESPONSIVE
═══════════════════════════════════════════════════════════════ */

/* Tablet: 2 columns */
@media (max-width: 1024px) {
  .status-hero-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Mobile: horizontal scroll-snap carousel */
@media (max-width: 640px) {
  .status-hero-grid {
    grid-template-columns: repeat(4, 78vw);
    overflow-x: auto;
    scroll-snap-type: x mandatory;
    -webkit-overflow-scrolling: touch;
    scrollbar-width: none;
    padding-bottom: 6px;
  }
  .status-hero-grid::-webkit-scrollbar { display: none; }
  .hero-card { scroll-snap-align: start; }
  .hc-count  { font-size: 2.25rem; }
}

/* Respect reduced-motion preference */
@media (prefers-reduced-motion: reduce) {
  .hero-card          { animation: none !important; opacity: 1; }
  .hero-card:hover    { transform: none; }
  .hero-card:active   { transform: none; }
  .hc-shimmer         { display: none; }
  .hc-bg-circle       { animation: none; }
  .hc-pulse__core     { animation: none; }
  .hc-pulse__ring     { animation: none; }
  .hc-icon            { animation: none; }
  .hc-bar-fill        { transition: none; }
  .table-swap-enter-active,
  .table-swap-leave-active { transition: none; }
}
</style>
