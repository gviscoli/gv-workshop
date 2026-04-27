<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div class="card budget-panel">
      <div class="card-header">
        <h3 class="card-title">{{ t('restocking.budgetSettings') }}</h3>
      </div>
      <div class="budget-form">
        <label class="budget-label">{{ t('restocking.budgetCeiling') }}</label>
        <input
          v-model.number="budgetInput"
          type="number"
          min="0"
          step="100"
          class="budget-input"
          :placeholder="t('restocking.budgetPlaceholder')"
        />
        <button @click="loadRecommendations" class="btn-primary">
          {{ t('restocking.getRecommendations') }}
        </button>
        <button v-if="budgetInput" @click="clearBudget" class="btn-secondary">
          {{ t('restocking.clearBudget') }}
        </button>
      </div>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>

      <div class="stats-grid" v-if="recommendations.length > 0">
        <div class="stat-card danger">
          <div class="stat-label">{{ t('restocking.itemsNeedingRestock') }}</div>
          <div class="stat-value">{{ recommendations.length }}</div>
        </div>
        <div class="stat-card warning">
          <div class="stat-label">{{ t('restocking.selectedItems') }}</div>
          <div class="stat-value">{{ selectedSkus.size }}</div>
        </div>
        <div class="stat-card info">
          <div class="stat-label">{{ t('restocking.selectedCost') }}</div>
          <div class="stat-value">${{ formatNumber(selectedTotalCost) }}</div>
        </div>
      </div>

      <div class="card" v-if="recommendations.length > 0">
        <div class="card-header">
          <h3 class="card-title">
            {{ t('restocking.recommendations') }} ({{ recommendations.length }})
          </h3>
          <div class="header-actions">
            <button @click="selectAll" class="btn-secondary">{{ t('restocking.selectAll') }}</button>
            <button @click="deselectAll" class="btn-secondary">{{ t('restocking.deselectAll') }}</button>
          </div>
        </div>

        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th></th>
                <th>{{ t('inventory.table.sku') }}</th>
                <th>{{ t('inventory.table.itemName') }}</th>
                <th>{{ t('inventory.table.category') }}</th>
                <th>{{ t('inventory.table.warehouse') }}</th>
                <th>{{ t('inventory.table.quantityOnHand') }}</th>
                <th>{{ t('inventory.table.reorderPoint') }}</th>
                <th>{{ t('demand.table.trend') }}</th>
                <th>{{ t('restocking.recommendedQty') }}</th>
                <th>{{ t('inventory.table.unitCost') }}</th>
                <th>{{ t('restocking.estimatedCost') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="item in recommendations"
                :key="item.sku"
                :class="{ 'row-selected': selectedSkus.has(item.sku) }"
                @click="toggleSelection(item.sku)"
                class="clickable-row"
              >
                <td>
                  <input
                    type="checkbox"
                    :checked="selectedSkus.has(item.sku)"
                    @click.stop="toggleSelection(item.sku)"
                  />
                </td>
                <td><strong>{{ item.sku }}</strong></td>
                <td>{{ item.name }}</td>
                <td>{{ item.category }}</td>
                <td>{{ item.warehouse }}</td>
                <td>
                  <span :class="['badge', item.quantity_on_hand <= item.reorder_point ? 'danger' : 'warning']">
                    {{ item.quantity_on_hand }}
                  </span>
                </td>
                <td>{{ item.reorder_point }}</td>
                <td>
                  <span v-if="item.trend" :class="['badge', item.trend]">
                    {{ t(`trends.${item.trend}`) }}
                  </span>
                  <span v-else class="badge stable">—</span>
                </td>
                <td><strong>{{ item.recommended_quantity }}</strong></td>
                <td>${{ item.unit_cost.toFixed(2) }}</td>
                <td><strong>${{ formatNumber(item.estimated_cost) }}</strong></td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="submit-bar" v-if="selectedSkus.size > 0">
          <span class="submit-summary">
            {{ selectedSkus.size }} {{ t('restocking.itemsSelected') }} —
            {{ t('restocking.totalCost') }}: <strong>${{ formatNumber(selectedTotalCost) }}</strong>
          </span>
          <button
            @click="submitPurchaseOrders"
            class="btn-primary"
            :disabled="submitting"
          >
            {{ submitting ? t('common.loading') : t('restocking.submitOrders') }}
          </button>
        </div>
      </div>

      <div class="card" v-else-if="hasSearched">
        <div class="empty-state">{{ t('restocking.noRecommendations') }}</div>
      </div>
    </div>

    <div v-if="submitSuccess" class="success-banner">
      {{ lastSubmittedCount }} {{ t('restocking.ordersSubmitted') }}
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t } = useI18n()
    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    const loading = ref(false)
    const error = ref(null)
    const recommendations = ref([])
    const budgetInput = ref(null)
    const selectedSkus = ref(new Set())
    const submitting = ref(false)
    const submitSuccess = ref(false)
    const lastSubmittedCount = ref(0)
    const hasSearched = ref(false)

    const selectedItems = computed(() =>
      recommendations.value.filter(r => selectedSkus.value.has(r.sku))
    )

    const selectedTotalCost = computed(() =>
      selectedItems.value.reduce((sum, item) => sum + item.estimated_cost, 0)
    )

    const loadRecommendations = async () => {
      try {
        loading.value = true
        error.value = null
        hasSearched.value = true
        selectedSkus.value = new Set()
        const filters = getCurrentFilters()
        recommendations.value = await api.getRestockingRecommendations({
          budget: budgetInput.value || null,
          warehouse: filters.warehouse,
          category: filters.category
        })
      } catch (err) {
        error.value = 'Failed to load recommendations: ' + err.message
      } finally {
        loading.value = false
      }
    }

    const toggleSelection = (sku) => {
      const next = new Set(selectedSkus.value)
      if (next.has(sku)) next.delete(sku)
      else next.add(sku)
      selectedSkus.value = next
    }

    const selectAll = () => {
      selectedSkus.value = new Set(recommendations.value.map(r => r.sku))
    }

    const deselectAll = () => {
      selectedSkus.value = new Set()
    }

    const clearBudget = () => {
      budgetInput.value = null
      loadRecommendations()
    }

    const formatNumber = (value) => {
      return Number(value).toLocaleString('en-US', {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2
      })
    }

    const submitPurchaseOrders = async () => {
      try {
        submitting.value = true
        const items = selectedItems.value.map(r => ({
          sku: r.sku,
          name: r.name,
          warehouse: r.warehouse,
          quantity: r.recommended_quantity,
          unit_cost: r.unit_cost
        }))
        await api.createPurchaseOrder({ items, notes: '' })
        lastSubmittedCount.value = items.length
        submitSuccess.value = true
        deselectAll()
        setTimeout(() => { submitSuccess.value = false }, 4000)
      } catch (err) {
        error.value = 'Failed to submit orders: ' + err.message
      } finally {
        submitting.value = false
      }
    }

    watch([selectedLocation, selectedCategory], () => {
      if (hasSearched.value) loadRecommendations()
    })

    return {
      t,
      loading,
      error,
      recommendations,
      budgetInput,
      selectedSkus,
      submitting,
      submitSuccess,
      lastSubmittedCount,
      hasSearched,
      selectedItems,
      selectedTotalCost,
      loadRecommendations,
      toggleSelection,
      selectAll,
      deselectAll,
      clearBudget,
      formatNumber,
      submitPurchaseOrders
    }
  }
}
</script>

<style scoped>
.budget-panel {
  margin-bottom: 1.25rem;
}

.budget-form {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.budget-label {
  font-weight: 600;
  font-size: 0.875rem;
  color: #475569;
}

.budget-input {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 0.5rem 0.875rem;
  font-size: 0.875rem;
  color: #0f172a;
  outline: none;
  width: 200px;
  transition: border-color 0.15s ease;
}

.budget-input:focus {
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
}

.btn-primary {
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 0.5rem 1.25rem;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s ease;
}

.btn-primary:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-primary:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.btn-secondary {
  background: #f1f5f9;
  color: #475569;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 0.5rem 1.25rem;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s ease;
}

.btn-secondary:hover {
  background: #e2e8f0;
}

.header-actions {
  display: flex;
  gap: 0.5rem;
}

.clickable-row {
  cursor: pointer;
}

.row-selected {
  background: #eff6ff !important;
}

.submit-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0.75rem;
  border-top: 1px solid #e2e8f0;
  background: #f8fafc;
  margin-top: 0.5rem;
  border-radius: 0 0 8px 8px;
}

.submit-summary {
  font-size: 0.875rem;
  color: #334155;
}

.empty-state {
  text-align: center;
  padding: 3rem;
  color: #64748b;
  font-size: 0.938rem;
}

.success-banner {
  background: #d1fae5;
  color: #065f46;
  border: 1px solid #6ee7b7;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  margin-top: 1rem;
  font-size: 0.938rem;
  font-weight: 500;
}
</style>
