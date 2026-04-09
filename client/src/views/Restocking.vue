<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>

      <!-- Budget Slider Card -->
      <div class="card" :style="{ '--anim-delay': '0ms' }">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.budgetLabel') }}</h3>
        </div>
        <div class="budget-section">
          <div class="budget-display">{{ formatCurrency(budget, currentCurrency) }}</div>
          <input
            type="range"
            v-model.number="budget"
            min="0"
            max="100000"
            step="1000"
            class="budget-slider"
          />
          <div class="budget-range-labels">
            <span>{{ formatCurrency(0, currentCurrency) }}</span>
            <span>{{ formatCurrency(100000, currentCurrency) }}</span>
          </div>
          <p class="budget-help">{{ t('restocking.budgetHelp') }}</p>
        </div>
      </div>

      <!-- Recommendations Card -->
      <div class="card" :style="{ '--anim-delay': '120ms' }">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.recommendationsTitle') }}</h3>
          <span class="card-meta">
            {{ t('restocking.itemsSelected', { count: recommendations.length }) }}
            &mdash; {{ t('restocking.totalCost') }}: {{ formatCurrency(totalCost, currentCurrency) }}
          </span>
        </div>

        <div v-if="recommendations.length === 0" class="no-data-message">
          {{ t('restocking.noRecommendations') }}
        </div>
        <div v-else class="table-container">
          <table class="restocking-table">
            <thead>
              <tr>
                <th>{{ t('restocking.table.sku') }}</th>
                <th>{{ t('restocking.table.itemName') }}</th>
                <th>{{ t('restocking.table.trend') }}</th>
                <th class="col-num">{{ t('restocking.table.currentDemand') }}</th>
                <th class="col-num">{{ t('restocking.table.forecastedDemand') }}</th>
                <th class="col-num">{{ t('restocking.table.quantityToOrder') }}</th>
                <th class="col-num">{{ t('restocking.table.unitCost') }}</th>
                <th class="col-num">{{ t('restocking.table.lineTotal') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendations" :key="item.sku">
                <td><strong>{{ item.sku }}</strong></td>
                <td>{{ item.item_name }}</td>
                <td>
                  <span :class="['badge', getTrendClass(item.trend)]">
                    {{ t(`trends.${item.trend}`) }}
                  </span>
                </td>
                <td class="col-num">{{ item.current_demand }}</td>
                <td class="col-num">{{ item.forecasted_demand }}</td>
                <td class="col-num"><strong>{{ item.quantity }}</strong></td>
                <td class="col-num">{{ formatCurrency(item.unit_cost, currentCurrency) }}</td>
                <td class="col-num"><strong>{{ formatCurrency(item.line_total, currentCurrency) }}</strong></td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="card-footer">
          <p class="delivery-note">{{ t('restocking.deliveryNote') }}</p>
          <button
            class="btn-primary"
            :disabled="recommendations.length === 0"
            @click="openPayment"
          >
            {{ t('restocking.placeOrder') }}
          </button>
        </div>
      </div>

      <!-- Success / Error banners -->
      <div v-if="successMessage" class="success-banner">{{ successMessage }}</div>
      <div v-if="orderError" class="error">{{ orderError }}</div>

      <!-- Payment Gateway Modal -->
      <teleport to="body">
        <transition name="modal-fade">
          <div v-if="showPayment" class="pg-overlay" @click.self="closePayment">
            <div class="pg-modal">

              <!-- Modal header -->
              <div class="pg-header">
                <div class="pg-header-text">
                  <h3 class="pg-title">Secure Checkout</h3>
                  <p class="pg-subtitle">Your connection is encrypted &amp; secure</p>
                </div>
                <button class="pg-close" @click="closePayment" aria-label="Close">
                  <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
                </button>
              </div>

              <!-- Amount pill -->
              <div class="pg-amount-pill">
                <span class="pg-amount-label">Total Amount</span>
                <span class="pg-amount-value">{{ formatCurrency(totalCost, currentCurrency) }}</span>
              </div>

              <!-- 3D flip credit card preview -->
              <div class="cc-scene" :class="{ 'cc-scene--flipped': focusedField === 'cvv' }">
                <div class="cc-card">
                  <div class="cc-front">
                    <div class="cc-front-top">
                      <div class="cc-chip">
                        <div class="cc-chip-line"></div>
                        <div class="cc-chip-line"></div>
                        <div class="cc-chip-line"></div>
                      </div>
                      <div class="cc-brand-badge">{{ cardBrand }}</div>
                    </div>
                    <div class="cc-number-display">{{ cardNumberDisplay }}</div>
                    <div class="cc-front-bottom">
                      <div>
                        <div class="cc-field-label">Card Holder</div>
                        <div class="cc-field-value">{{ form.name.toUpperCase() || 'FULL NAME' }}</div>
                      </div>
                      <div>
                        <div class="cc-field-label">Expires</div>
                        <div class="cc-field-value">{{ form.expiry || 'MM/YY' }}</div>
                      </div>
                    </div>
                  </div>
                  <div class="cc-back">
                    <div class="cc-back-stripe"></div>
                    <div class="cc-cvv-row">
                      <div class="cc-cvv-label">CVV</div>
                      <div class="cc-cvv-box">{{ form.cvv ? '•'.repeat(form.cvv.length) : '•••' }}</div>
                    </div>
                  </div>
                </div>
              </div>

              <!-- Payment form -->
              <div class="pg-form" v-if="!paymentSuccess">
                <div class="pg-field">
                  <label class="pg-label">Card Number</label>
                  <input
                    class="pg-input"
                    :value="form.cardNumber"
                    @input="onCardNumberInput"
                    placeholder="0000 0000 0000 0000"
                    maxlength="19"
                    inputmode="numeric"
                    @focus="focusedField = 'number'"
                    @blur="focusedField = ''"
                  />
                </div>

                <div class="pg-field">
                  <label class="pg-label">Cardholder Name</label>
                  <input
                    class="pg-input"
                    v-model="form.name"
                    placeholder="John Doe"
                    @focus="focusedField = 'name'"
                    @blur="focusedField = ''"
                  />
                </div>

                <div class="pg-row">
                  <div class="pg-field">
                    <label class="pg-label">Expiry Date</label>
                    <input
                      class="pg-input"
                      :value="form.expiry"
                      @input="onExpiryInput"
                      placeholder="MM / YY"
                      maxlength="7"
                      inputmode="numeric"
                      @focus="focusedField = 'expiry'"
                      @blur="focusedField = ''"
                    />
                  </div>
                  <div class="pg-field">
                    <label class="pg-label">CVV</label>
                    <input
                      class="pg-input"
                      v-model="form.cvv"
                      type="password"
                      placeholder="•••"
                      maxlength="4"
                      inputmode="numeric"
                      @focus="focusedField = 'cvv'"
                      @blur="focusedField = ''"
                    />
                  </div>
                </div>

                <button
                  class="pg-pay-btn"
                  :class="{ 'pg-pay-btn--processing': paymentProcessing }"
                  :disabled="!isFormValid || paymentProcessing"
                  @click="submitPayment"
                >
                  <span v-if="!paymentProcessing">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"><rect x="1" y="4" width="22" height="16" rx="2" ry="2"/><line x1="1" y1="10" x2="23" y2="10"/></svg>
                    &nbsp; Pay {{ formatCurrency(totalCost, currentCurrency) }}
                  </span>
                  <span v-else class="pg-processing-text">
                    <span class="pg-spinner"></span>
                    Processing payment...
                  </span>
                </button>

                <div class="pg-security-note">
                  <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
                  256-bit SSL encrypted &nbsp;·&nbsp; PCI DSS compliant
                </div>
              </div>

              <!-- Success state -->
              <div v-else class="pg-success">
                <div class="pg-success-icon">
                  <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="20 6 9 17 4 12"/></svg>
                </div>
                <h4 class="pg-success-title">Payment Successful!</h4>
                <p class="pg-success-msg">Your restocking order has been confirmed. Items will be delivered within 14 days.</p>
              </div>

            </div>
          </div>
        </transition>
      </teleport>

    </div>
  </div>
</template>

<script>
import { ref, reactive, computed, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'
import { formatCurrency } from '../utils/currency'

export default {
  name: 'Restocking',
  setup() {
    const { t, currentCurrency } = useI18n()

    const loading = ref(true)
    const error = ref(null)
    const budget = ref(10000)
    const forecasts = ref([])
    const inventoryMap = ref({})
    const submitting = ref(false)

    // Payment gateway state
    const showPayment = ref(false)
    const paymentProcessing = ref(false)
    const paymentSuccess = ref(false)
    const focusedField = ref('')
    const form = reactive({ cardNumber: '', name: '', expiry: '', cvv: '' })
    const successMessage = ref(null)
    const orderError = ref(null)

    const loadData = async () => {
      try {
        loading.value = true
        error.value = null
        const [forecastData, inventoryData] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory()
        ])
        forecasts.value = forecastData
        // Build SKU -> unit_cost lookup map
        inventoryMap.value = Object.fromEntries(
          inventoryData.map(item => [item.sku, item.unit_cost])
        )
      } catch (err) {
        error.value = 'Failed to load data: ' + err.message
      } finally {
        loading.value = false
      }
    }

    const recommendations = computed(() => {
      // Filter: only items with positive demand gap, exclude decreasing trend
      const sorted = forecasts.value
        .filter(f => f.trend !== 'decreasing' && f.forecasted_demand > f.current_demand)
        .sort((a, b) => {
          const order = { increasing: 0, stable: 1 }
          return (order[a.trend] ?? 2) - (order[b.trend] ?? 2)
        })

      // Greedy fill within budget
      let remaining = budget.value
      return sorted.reduce((result, f) => {
        const unitCost = inventoryMap.value[f.item_sku]
        if (unitCost === undefined) return result
        const qty = f.forecasted_demand - f.current_demand
        const lineTotal = qty * unitCost
        if (lineTotal <= remaining) {
          result.push({
            sku: f.item_sku,
            item_name: f.item_name,
            trend: f.trend,
            current_demand: f.current_demand,
            forecasted_demand: f.forecasted_demand,
            quantity: qty,
            unit_cost: unitCost,
            line_total: lineTotal
          })
          remaining -= lineTotal
        }
        return result
      }, [])
    })

    const totalCost = computed(() =>
      recommendations.value.reduce((sum, item) => sum + item.line_total, 0)
    )

    const getTrendClass = (trend) => {
      return trend === 'increasing' ? 'success' : 'info'
    }

    // Card brand detection
    const cardBrand = computed(() => {
      const n = form.cardNumber.replace(/\s/g, '')
      if (n.startsWith('4')) return 'VISA'
      if (/^5[1-5]/.test(n) || /^2[2-7]/.test(n)) return 'MC'
      if (/^3[47]/.test(n)) return 'AMEX'
      return 'CARD'
    })

    // Card number display on the card face (padded with bullets)
    const cardNumberDisplay = computed(() => {
      const raw = form.cardNumber.replace(/\s/g, '').padEnd(16, '•')
      return `${raw.slice(0,4)} ${raw.slice(4,8)} ${raw.slice(8,12)} ${raw.slice(12,16)}`
    })

    // Form validation
    const isFormValid = computed(() => {
      const raw = form.cardNumber.replace(/\s/g, '')
      return (
        raw.length === 16 &&
        form.name.trim().length >= 2 &&
        /^\d{2} \/ \d{2}$/.test(form.expiry) &&
        form.cvv.length >= 3
      )
    })

    // Input formatters
    const onCardNumberInput = (e) => {
      const raw = e.target.value.replace(/\D/g, '').slice(0, 16)
      form.cardNumber = raw.replace(/(.{4})/g, '$1 ').trim()
    }

    const onExpiryInput = (e) => {
      const raw = e.target.value.replace(/\D/g, '').slice(0, 4)
      if (raw.length >= 2) {
        form.expiry = raw.slice(0, 2) + ' / ' + raw.slice(2)
      } else {
        form.expiry = raw
      }
    }

    // Open/close
    const openPayment = () => {
      if (recommendations.value.length === 0) return
      showPayment.value = true
      paymentSuccess.value = false
      paymentProcessing.value = false
      Object.assign(form, { cardNumber: '', name: '', expiry: '', cvv: '' })
    }

    const closePayment = () => {
      if (paymentProcessing.value) return
      showPayment.value = false
    }

    // Rename old placeOrder logic to confirmOrder
    const confirmOrder = async () => {
      submitting.value = true
      orderError.value = null
      try {
        const deliveryDate = new Date()
        deliveryDate.setDate(deliveryDate.getDate() + 14)
        const expectedDelivery = deliveryDate.toISOString().split('T')[0]
        await api.createRestockingOrder({
          budget: budget.value,
          total_cost: totalCost.value,
          expected_delivery_date: expectedDelivery,
          items: recommendations.value.map(({ sku, item_name, quantity, unit_cost, line_total }) => ({
            sku, item_name, quantity, unit_cost, line_total
          }))
        })
        successMessage.value = t('restocking.orderSuccess')
      } catch (err) {
        orderError.value = t('restocking.orderError')
        console.error('Failed to place restocking order:', err)
      } finally {
        submitting.value = false
      }
    }

    // Payment submission
    const submitPayment = async () => {
      if (!isFormValid.value) return
      paymentProcessing.value = true
      // Simulate payment processing (2.5s)
      await new Promise(r => setTimeout(r, 2500))
      paymentProcessing.value = false
      paymentSuccess.value = true
      // Wait 1.8s showing success, then submit order and close
      await new Promise(r => setTimeout(r, 1800))
      await confirmOrder()
      showPayment.value = false
    }

    onMounted(loadData)

    return {
      t,
      currentCurrency,
      loading,
      error,
      budget,
      recommendations,
      totalCost,
      submitting,
      getTrendClass,
      formatCurrency,
      // Payment gateway
      showPayment,
      paymentProcessing,
      paymentSuccess,
      focusedField,
      form,
      cardBrand,
      cardNumberDisplay,
      isFormValid,
      onCardNumberInput,
      onExpiryInput,
      openPayment,
      closePayment,
      submitPayment,
      successMessage,
      orderError
    }
  }
}
</script>

<style scoped>
@keyframes cardEnter {
  from { opacity: 0; transform: translateY(28px) scale(0.96); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

.card {
  animation: cardEnter 0.52s cubic-bezier(0.34, 1.56, 0.64, 1) var(--anim-delay, 0s) both;
  transition: transform 0.22s ease, box-shadow 0.22s ease;
}

.card:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.1);
}

@media (prefers-reduced-motion: reduce) {
  .card { animation: none; }
}

.budget-section {
  padding: 0.5rem 0 1rem;
}

.budget-display {
  font-size: 2rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 1rem;
}

.budget-slider {
  width: 100%;
  height: 6px;
  accent-color: #2563eb;
  cursor: pointer;
  margin-bottom: 0.5rem;
}

.budget-range-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #94a3b8;
  margin-bottom: 0.5rem;
}

.budget-help {
  font-size: 0.813rem;
  color: #64748b;
}

.card-meta {
  font-size: 0.875rem;
  color: #64748b;
  font-weight: 500;
}

.restocking-table {
  width: 100%;
  table-layout: fixed;
}

.col-num {
  width: 110px;
  text-align: right;
}

.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0 0;
  margin-top: 1rem;
  border-top: 1px solid #e2e8f0;
}

.delivery-note {
  font-size: 0.813rem;
  color: #64748b;
}

.btn-primary {
  background: #2563eb;
  color: white;
  border: none;
  padding: 0.625rem 1.5rem;
  border-radius: 6px;
  font-size: 0.938rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s ease;
  white-space: nowrap;
}

.btn-primary:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-primary:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.no-data-message {
  padding: 2rem;
  text-align: center;
  color: #64748b;
  font-size: 0.938rem;
}

.success-banner {
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  color: #065f46;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  margin-top: 1rem;
  font-size: 0.938rem;
  font-weight: 500;
}

/* ── Payment Gateway Overlay ─────────────────────────────── */
.pg-overlay {
  position: fixed;
  inset: 0;
  background: rgba(10, 8, 30, 0.82);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

.pg-modal {
  width: 100%;
  max-width: 440px;
  background: linear-gradient(145deg, #0f0c29 0%, #302b63 55%, #1a1a3e 100%);
  border-radius: 20px;
  border: 1px solid rgba(255,255,255,0.09);
  box-shadow: 0 40px 100px rgba(0,0,0,0.65), 0 0 0 1px rgba(255,255,255,0.04), inset 0 1px 0 rgba(255,255,255,0.08);
  padding: 1.75rem 1.75rem 1.5rem;
  position: relative;
  overflow: hidden;
}

/* Subtle radial glow inside modal */
.pg-modal::before {
  content: '';
  position: absolute;
  top: -80px;
  right: -80px;
  width: 300px;
  height: 300px;
  background: radial-gradient(circle, rgba(139,92,246,0.18) 0%, transparent 70%);
  pointer-events: none;
  border-radius: 50%;
}

/* ── Header ──────────────────────────────────────────────── */
.pg-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  margin-bottom: 1.25rem;
}

.pg-title {
  font-size: 1.1rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.02em;
  margin-bottom: 2px;
}

.pg-subtitle {
  font-size: 0.72rem;
  color: rgba(255,255,255,0.45);
  letter-spacing: 0.02em;
}

.pg-close {
  background: rgba(255,255,255,0.08);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 8px;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: rgba(255,255,255,0.6);
  transition: background 0.15s, color 0.15s;
  flex-shrink: 0;
}

.pg-close:hover { background: rgba(255,255,255,0.14); color: #fff; }

/* ── Amount pill ─────────────────────────────────────────── */
.pg-amount-pill {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 12px;
  padding: 0.75rem 1.125rem;
  margin-bottom: 1.25rem;
}

.pg-amount-label {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: rgba(255,255,255,0.45);
}

.pg-amount-value {
  font-size: 1.25rem;
  font-weight: 800;
  color: #f8d57e;
  letter-spacing: -0.02em;
}

/* ── 3D Credit Card ──────────────────────────────────────── */
.cc-scene {
  perspective: 1200px;
  height: 170px;
  margin-bottom: 1.5rem;
}

.cc-card {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.65s cubic-bezier(0.4, 0, 0.2, 1);
  border-radius: 16px;
}

.cc-scene--flipped .cc-card {
  transform: rotateY(180deg);
}

.cc-front,
.cc-back {
  position: absolute;
  inset: 0;
  border-radius: 16px;
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  padding: 1.25rem 1.375rem;
  box-shadow: 0 16px 40px rgba(0,0,0,0.45);
}

.cc-front {
  background: linear-gradient(135deg, #1a1a3e 0%, #16213e 45%, #0f3460 100%);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  border: 1px solid rgba(255,255,255,0.08);
}

.cc-back {
  background: linear-gradient(135deg, #0f3460 0%, #16213e 55%, #1a1a3e 100%);
  transform: rotateY(180deg);
  border: 1px solid rgba(255,255,255,0.08);
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.cc-front-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

/* Gold chip */
.cc-chip {
  width: 40px;
  height: 30px;
  background: linear-gradient(135deg, #c9a84c 0%, #f5d17a 45%, #c9a84c 100%);
  border-radius: 5px;
  border: 1px solid rgba(255,255,255,0.25);
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 4px;
  padding: 5px 6px;
}

.cc-chip-line {
  height: 2px;
  background: rgba(100,70,0,0.4);
  border-radius: 1px;
}

.cc-brand-badge {
  font-size: 0.7rem;
  font-weight: 800;
  letter-spacing: 0.12em;
  color: rgba(255,255,255,0.7);
  border: 1px solid rgba(255,255,255,0.2);
  padding: 3px 8px;
  border-radius: 4px;
  background: rgba(255,255,255,0.06);
}

.cc-number-display {
  font-family: 'Courier New', monospace;
  font-size: 1.1rem;
  letter-spacing: 0.2em;
  color: rgba(255,255,255,0.9);
  text-align: center;
  font-weight: 600;
  text-shadow: 0 1px 4px rgba(0,0,0,0.4);
}

.cc-front-bottom {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
}

.cc-field-label {
  font-size: 0.58rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: rgba(255,255,255,0.42);
  margin-bottom: 2px;
}

.cc-field-value {
  font-size: 0.78rem;
  font-weight: 600;
  color: rgba(255,255,255,0.88);
  letter-spacing: 0.04em;
  white-space: nowrap;
  overflow: hidden;
  max-width: 160px;
  text-overflow: ellipsis;
}

.cc-back-stripe {
  height: 44px;
  background: rgba(0,0,0,0.6);
  margin: 0 -1.375rem;
  margin-bottom: 1rem;
}

.cc-cvv-row {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 4px;
}

.cc-cvv-label {
  font-size: 0.6rem;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: rgba(255,255,255,0.4);
}

.cc-cvv-box {
  background: rgba(255,255,255,0.92);
  color: #1a1a2e;
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
  font-weight: 700;
  padding: 5px 12px;
  border-radius: 4px;
  letter-spacing: 0.18em;
  min-width: 60px;
  text-align: center;
}

/* ── Form fields ─────────────────────────────────────────── */
.pg-form { display: flex; flex-direction: column; gap: 0.875rem; }

.pg-field { display: flex; flex-direction: column; gap: 5px; }

.pg-row { display: grid; grid-template-columns: 1fr 1fr; gap: 0.75rem; }

.pg-label {
  font-size: 0.72rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: rgba(255,255,255,0.5);
}

.pg-input {
  background: rgba(255,255,255,0.07);
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 10px;
  padding: 0.625rem 0.875rem;
  font-size: 0.9rem;
  color: #ffffff;
  outline: none;
  transition: border-color 0.2s, background 0.2s, box-shadow 0.2s;
  width: 100%;
}

.pg-input::placeholder { color: rgba(255,255,255,0.25); }

.pg-input:focus {
  border-color: rgba(139,92,246,0.7);
  background: rgba(255,255,255,0.1);
  box-shadow: 0 0 0 3px rgba(139,92,246,0.18);
}

/* ── Pay button ──────────────────────────────────────────── */
.pg-pay-btn {
  width: 100%;
  padding: 0.875rem;
  border: none;
  border-radius: 12px;
  font-size: 0.9375rem;
  font-weight: 700;
  cursor: pointer;
  letter-spacing: 0.02em;
  background: linear-gradient(135deg, #f59e0b 0%, #d97706 50%, #b45309 100%);
  color: #1a0a00;
  box-shadow: 0 6px 24px rgba(245,158,11,0.38), 0 2px 8px rgba(0,0,0,0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  transition: transform 0.15s ease, box-shadow 0.15s ease, opacity 0.15s ease;
  margin-top: 0.25rem;
}

.pg-pay-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 32px rgba(245,158,11,0.48), 0 4px 12px rgba(0,0,0,0.25);
}

.pg-pay-btn:active:not(:disabled) { transform: translateY(0); }

.pg-pay-btn:disabled {
  opacity: 0.38;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.pg-pay-btn--processing {
  background: linear-gradient(135deg, #6d28d9 0%, #4c1d95 100%);
  color: #fff;
  box-shadow: 0 6px 24px rgba(109,40,217,0.38);
}

/* Spinner */
.pg-processing-text { display: flex; align-items: center; gap: 0.625rem; }

.pg-spinner {
  width: 16px;
  height: 16px;
  border: 2.5px solid rgba(255,255,255,0.25);
  border-top-color: #ffffff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  display: inline-block;
  flex-shrink: 0;
}

@keyframes spin { to { transform: rotate(360deg); } }

/* ── Security note ───────────────────────────────────────── */
.pg-security-note {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
  font-size: 0.67rem;
  color: rgba(255,255,255,0.3);
  letter-spacing: 0.04em;
  margin-top: 0.25rem;
}

/* ── Success state ───────────────────────────────────────── */
.pg-success {
  text-align: center;
  padding: 1.5rem 0 0.5rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
  animation: successFadeIn 0.5s ease forwards;
}

.pg-success-icon {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  background: linear-gradient(135deg, #059669, #10b981);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  box-shadow: 0 8px 32px rgba(16,185,129,0.45);
  animation: successPop 0.5s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

.pg-success-title {
  font-size: 1.1rem;
  font-weight: 700;
  color: #ffffff;
}

.pg-success-msg {
  font-size: 0.8rem;
  color: rgba(255,255,255,0.5);
  max-width: 280px;
  line-height: 1.5;
}

@keyframes successPop {
  0%   { transform: scale(0.3); opacity: 0; }
  100% { transform: scale(1);   opacity: 1; }
}

@keyframes successFadeIn {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* ── Modal transition ────────────────────────────────────── */
.modal-fade-enter-active { transition: opacity 0.28s ease, transform 0.28s cubic-bezier(0.34, 1.56, 0.64, 1); }
.modal-fade-leave-active { transition: opacity 0.18s ease, transform 0.18s ease; }
.modal-fade-enter-from   { opacity: 0; transform: scale(0.88); }
.modal-fade-leave-to     { opacity: 0; transform: scale(0.94); }
</style>
