<template>
  <div class="backlog">
    <div class="page-header">
      <h2>Backlog Management</h2>
      <p>Track and resolve inventory shortages</p>
    </div>

    <div v-if="loading" class="loading">Loading backlog...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <div class="stats-grid">
        <div class="stat-card danger" :style="{ '--anim-delay': '0ms' }">
          <div class="stat-label">High Priority</div>
          <div class="stat-value">{{ animatedCounts.high }}</div>
        </div>
        <div class="stat-card warning" :style="{ '--anim-delay': '90ms' }">
          <div class="stat-label">Medium Priority</div>
          <div class="stat-value">{{ animatedCounts.medium }}</div>
        </div>
        <div class="stat-card info" :style="{ '--anim-delay': '180ms' }">
          <div class="stat-label">Low Priority</div>
          <div class="stat-value">{{ animatedCounts.low }}</div>
        </div>
        <div class="stat-card" :style="{ '--anim-delay': '270ms' }">
          <div class="stat-label">Total Backlog Items</div>
          <div class="stat-value">{{ animatedCounts.total }}</div>
        </div>
      </div>

      <div class="card">
        <div class="card-header">
          <h3 class="card-title">Backlog Items</h3>
        </div>
        <div v-if="backlogItems.length === 0" style="padding: 3rem; text-align: center;">
          <p style="font-size: 1.125rem; color: #10b981; font-weight: 600;">
            ✓ No backlog items - all orders can be fulfilled!
          </p>
        </div>
        <div v-else class="table-container">
          <table>
            <thead>
              <tr>
                <th>Order ID</th>
                <th>SKU</th>
                <th>Item Name</th>
                <th>Quantity Needed</th>
                <th>Quantity Available</th>
                <th>Shortage</th>
                <th>Days Delayed</th>
                <th>Priority</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in backlogItems" :key="item.id">
                <td><strong>{{ item.order_id }}</strong></td>
                <td><strong>{{ item.item_sku }}</strong></td>
                <td>{{ item.item_name }}</td>
                <td>{{ item.quantity_needed }}</td>
                <td>{{ item.quantity_available }}</td>
                <td>
                  <span class="badge danger">
                    {{ item.quantity_needed - item.quantity_available }} units short
                  </span>
                </td>
                <td>
                  <span :style="{ color: item.days_delayed > 7 ? '#ef4444' : '#f59e0b' }">
                    {{ item.days_delayed }} days
                  </span>
                </td>
                <td>
                  <span :class="['badge', item.priority]">
                    {{ item.priority }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch, computed, reactive, nextTick } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'

export default {
  name: 'Backlog',
  setup() {
    const loading = ref(true)
    const error = ref(null)
    const allBacklogItems = ref([])
    const inventoryItems = ref([])

    // Use shared filters
    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    // Filter backlog based on inventory filters
    const backlogItems = computed(() => {
      if (selectedLocation.value === 'all' && selectedCategory.value === 'all') {
        return allBacklogItems.value
      }

      // Get SKUs of items that match the filters
      const validSkus = new Set(inventoryItems.value.map(item => item.sku))
      return allBacklogItems.value.filter(b => validSkus.has(b.item_sku))
    })

    const loadBacklog = async () => {
      try {
        loading.value = true
        const filters = getCurrentFilters()

        const [backlogData, inventoryData] = await Promise.all([
          api.getBacklog(),
          api.getInventory({
            warehouse: filters.warehouse,
            category: filters.category
          })
        ])

        allBacklogItems.value = backlogData
        inventoryItems.value = inventoryData
      } catch (err) {
        error.value = 'Failed to load backlog: ' + err.message
      } finally {
        loading.value = false
      }
      await nextTick()
      triggerCounts()
    }

    const getBacklogByPriority = (priority) => {
      return backlogItems.value.filter(item => item.priority === priority)
    }

    // Animated counts for stat cards
    const animatedCounts = reactive({ high: 0, medium: 0, low: 0, total: 0 })

    const easeOutCubic = (x) => 1 - Math.pow(1 - x, 3)

    const animateCount = (key, target) => {
      const DURATION = 1200
      const t0 = performance.now()
      const tick = (now) => {
        const p = Math.min((now - t0) / DURATION, 1)
        animatedCounts[key] = Math.round(easeOutCubic(p) * target)
        if (p < 1) requestAnimationFrame(tick)
      }
      requestAnimationFrame(tick)
    }

    const triggerCounts = () => {
      const targets = [
        ['high',   getBacklogByPriority('high').length],
        ['medium', getBacklogByPriority('medium').length],
        ['low',    getBacklogByPriority('low').length],
        ['total',  backlogItems.value.length],
      ]
      targets.forEach(([key, val], i) => setTimeout(() => animateCount(key, val), i * 90))
    }

    // Watch for filter changes and reload data
    watch([selectedLocation, selectedCategory], loadBacklog)

    onMounted(loadBacklog)

    return {
      loading,
      error,
      backlogItems,
      getBacklogByPriority,
      animatedCounts
    }
  }
}
</script>

<style scoped>
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

@keyframes cardEnter {
  from { opacity: 0; transform: translateY(28px) scale(0.96); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

.stat-card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 1.5rem 1.75rem;
  overflow: hidden;
  position: relative;
  animation: cardEnter 0.52s cubic-bezier(0.34, 1.56, 0.64, 1) var(--anim-delay, 0s) both;
  transition: transform 0.22s ease, box-shadow 0.22s ease;
}

.stat-card::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, rgba(255,255,255,0.6) 0%, transparent 60%);
  pointer-events: none;
}

.stat-card:hover {
  transform: translateY(-4px) scale(1.02);
  box-shadow: 0 14px 36px rgba(0, 0, 0, 0.12);
}

.stat-card.danger  { border-left: 4px solid #ef4444; }
.stat-card.warning { border-left: 4px solid #f59e0b; }
.stat-card.info    { border-left: 4px solid #3b82f6; }
.stat-card:not(.danger):not(.warning):not(.info) { border-left: 4px solid #8b5cf6; }

.stat-label {
  font-size: 0.8125rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.stat-value {
  font-size: 2.5rem;
  font-weight: 800;
  color: #0f172a;
  font-variant-numeric: tabular-nums;
  line-height: 1;
}

@media (prefers-reduced-motion: reduce) {
  .stat-card { animation: none; }
}
</style>
