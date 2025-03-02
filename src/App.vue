<template>
  <div class="min-h-screen bg-gray-100 text-gray-900">
    <header class="bg-emerald-600 text-white p-4 shadow-md">
      <h1 class="text-xl font-bold text-center">Simple Shopping</h1>
    </header>

    <main class="container mx-auto p-4 max-w-md">
      <!-- Currency Selection -->
      <div class="bg-white p-4 rounded-lg shadow mb-6">
        <label for="currency" class="block text-sm font-medium text-gray-700 mb-1">Select Currency</label>
        <select id="currency" v-model="selectedCurrency"
          class="w-full p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500">
          <option v-for="currency in currencies" :key="currency.code" :value="currency">
            {{ currency.code }} ({{ currency.symbol }})
          </option>
        </select>
      </div>

      <!-- Navigation Tabs -->
      <div class="flex mb-6 bg-white rounded-lg shadow overflow-hidden">
        <button v-for="tab in tabs" :key="tab.id" @click="activeTab = tab.id" :class="[
          'flex-1 py-3 text-center transition-colors',
          activeTab === tab.id
            ? 'bg-emerald-600 text-white font-medium'
            : 'bg-white text-gray-700 hover:bg-gray-100'
        ]">
          {{ tab.name }}
        </button>
      </div>

      <!-- Shopping Lists Tab -->
      <div v-if="activeTab === 'lists'" class="space-y-6">
        <div class="bg-white p-4 rounded-lg shadow">
          <h2 class="text-lg font-semibold mb-4">Create New Shopping List</h2>
          <div class="space-y-3">
            <div>
              <label for="listName" class="block text-sm font-medium text-gray-700 mb-1">List Name</label>
              <input id="listName" v-model="newList.name" type="text"
                class="w-full p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500"
                placeholder="Weekly Groceries" />
            </div>
            <div>
              <label for="listBudget" class="block text-sm font-medium text-gray-700 mb-1">Budget</label>
              <input id="listBudget" v-model="newList.budget" type="number"
                class="w-full p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500"
                placeholder="100.00" min="0" step="0.01" />
            </div>
            <button @click="createList"
              class="w-full bg-emerald-600 text-white py-2 rounded hover:bg-emerald-700 transition-colors"
              :disabled="!newList.name || !newList.budget">
              Create List
            </button>
          </div>
        </div>

        <div v-if="shoppingLists.length > 0" class="space-y-4">
          <h2 class="text-lg font-semibold">Your Shopping Lists</h2>
          <div v-for="list in shoppingLists" :key="list.id" class="bg-white p-4 rounded-lg shadow">
            <div class="flex justify-between items-center mb-2">
              <h3 class="font-medium">{{ list.name }}</h3>
              <span class="text-sm bg-gray-200 px-2 py-1 rounded">
                Budget: {{ formatCurrency(list.budget) }}
              </span>
            </div>
            <div class="flex justify-between text-sm text-gray-600 mb-3">
              <span>Items: {{ list.items.length }}</span>
              <span>
                Remaining: {{ formatCurrency(calculateRemaining(list)) }}
              </span>
            </div>
            <button @click="selectList(list)"
              class="w-full bg-emerald-100 text-emerald-800 py-2 rounded hover:bg-emerald-200 transition-colors">
              View Details
            </button>
          </div>
        </div>

        <div v-else class="bg-white p-4 rounded-lg shadow text-center text-gray-500">
          <p>You don't have any shopping lists yet.</p>
        </div>
      </div>

      <!-- List Details -->
      <div v-if="activeTab === 'details' && selectedList" class="space-y-6">
        <div class="bg-white p-4 rounded-lg shadow">
          <div class="flex justify-between items-center mb-4">
            <h2 class="text-lg font-semibold">{{ selectedList.name }}</h2>
            <button @click="backToLists" class="text-sm text-emerald-600">
              Back to Lists
            </button>
          </div>

          <div class="flex justify-between mb-4 text-sm">
            <span class="bg-emerald-100 text-emerald-800 px-2 py-1 rounded">
              Budget: {{ formatCurrency(selectedList.budget) }}
            </span>
            <span class="bg-gray-100 text-gray-800 px-2 py-1 rounded">
              Remaining: {{ formatCurrency(calculateRemaining(selectedList)) }}
            </span>
          </div>

          <div class="space-y-3 mb-4">
            <h3 class="font-medium">Add New Item</h3>
            <div>
              <label for="itemName" class="block text-sm font-medium text-gray-700 mb-1">Item Name</label>
              <input id="itemName" v-model="newItem.name" type="text"
                class="w-full p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500"
                placeholder="Milk" />
            </div>
            <div>
              <label for="itemCategory" class="block text-sm font-medium text-gray-700 mb-1">Category</label>
              <select id="itemCategory" v-model="newItem.categoryId"
                class="w-full p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500">
                <option value="">Select a category</option>
                <option v-for="category in categories" :key="category.id" :value="category.id">
                  {{ category.name }}
                </option>
              </select>
            </div>
            <div>
              <label for="itemPrice" class="block text-sm font-medium text-gray-700 mb-1">Expected Price</label>
              <input id="itemPrice" v-model="newItem.expectedPrice" type="number"
                class="w-full p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500"
                placeholder="2.99" min="0" step="0.01" />
            </div>
            <button @click="addItemToList"
              class="w-full bg-emerald-600 text-white py-2 rounded hover:bg-emerald-700 transition-colors"
              :disabled="!newItem.name || !newItem.expectedPrice">
              Add Item
            </button>
          </div>

          <div v-if="selectedList.items.length > 0" class="space-y-3">
            <h3 class="font-medium">Items</h3>
            <div v-for="item in selectedList.items" :key="item.id" class="border rounded p-3"
              :class="item.purchased ? 'bg-gray-50' : ''">
              <div class="flex justify-between items-center mb-2">
                <div>
                  <span class="font-medium">{{ item.name }}</span>
                  <span v-if="item.categoryId" class="ml-2 text-xs bg-gray-200 px-1.5 py-0.5 rounded">
                    {{ getCategoryName(item.categoryId) }}
                  </span>
                </div>
                <div class="text-sm">
                  <span v-if="!item.purchased" class="text-emerald-600">
                    Expected: {{ formatCurrency(item.expectedPrice) }}
                  </span>
                  <span v-else class="text-gray-600">
                    Paid: {{ formatCurrency(item.actualPrice) }}
                  </span>
                </div>
              </div>

              <div class="mt-2 flex space-x-2">
                <button @click="openPurchaseModal(item)"
                  class="flex-1 bg-emerald-100 text-emerald-800 py-1.5 rounded text-sm hover:bg-emerald-200 transition-colors">
                  {{ item.purchased ? 'Edit' : 'Mark as Purchased' }}
                </button>
                <button v-if="item.purchased" @click="unpurchaseItem(item)"
                  class="flex-1 bg-red-100 text-red-800 py-1.5 rounded text-sm hover:bg-red-200 transition-colors">
                  Unpurchase
                </button>
              </div>
              <div v-if="item.purchased" class="text-sm text-gray-600 mt-1">
                Purchased at: {{ getStoreName(item.storeId) }}
              </div>
            </div>
          </div>

          <div v-else class="text-center text-gray-500 my-4">
            <p>No items in this list yet.</p>
          </div>
        </div>
      </div>

      <!-- Categories Tab -->
      <div v-if="activeTab === 'categories'" class="space-y-6">
        <div class="bg-white p-4 rounded-lg shadow">
          <h2 class="text-lg font-semibold mb-4">Add New Category</h2>
          <div class="flex space-x-2">
            <input v-model="newCategory" type="text"
              class="flex-1 p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500"
              placeholder="Dairy" />
            <button @click="addCategory"
              class="bg-emerald-600 text-white px-4 py-2 rounded hover:bg-emerald-700 transition-colors"
              :disabled="!newCategory">
              Add
            </button>
          </div>
        </div>

        <div v-if="categories.length > 0" class="bg-white p-4 rounded-lg shadow">
          <h2 class="text-lg font-semibold mb-4">Your Categories</h2>
          <ul class="divide-y">
            <li v-for="category in categories" :key="category.id" class="py-2 flex justify-between items-center">
              <span>{{ category.name }}</span>
              <button @click="removeCategory(category.id)" class="text-red-600 hover:text-red-800">
                Remove
              </button>
            </li>
          </ul>
        </div>

        <div v-else class="bg-white p-4 rounded-lg shadow text-center text-gray-500">
          <p>You don't have any categories yet.</p>
        </div>
      </div>

      <!-- Stores Tab -->
      <div v-if="activeTab === 'stores'" class="space-y-6">
        <div class="bg-white p-4 rounded-lg shadow">
          <h2 class="text-lg font-semibold mb-4">Add New Store</h2>
          <div class="flex space-x-2">
            <input v-model="newStore" type="text"
              class="flex-1 p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500"
              placeholder="Walmart" />
            <button @click="addStore"
              class="bg-emerald-600 text-white px-4 py-2 rounded hover:bg-emerald-700 transition-colors"
              :disabled="!newStore">
              Add
            </button>
          </div>
        </div>

        <div v-if="stores.length > 0" class="bg-white p-4 rounded-lg shadow">
          <h2 class="text-lg font-semibold mb-4">Your Stores</h2>
          <ul class="divide-y">
            <li v-for="store in stores" :key="store.id" class="py-2 flex justify-between items-center">
              <span>{{ store.name }}</span>
              <button @click="removeStore(store.id)" class="text-red-600 hover:text-red-800">
                Remove
              </button>
            </li>
          </ul>
        </div>

        <div v-else class="bg-white p-4 rounded-lg shadow text-center text-gray-500">
          <p>You don't have any stores yet.</p>
        </div>
      </div>
    </main>

    <!-- Purchase Modal -->
    <div v-if="purchaseModal.show"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
      <div class="bg-white rounded-lg shadow-lg p-6 w-full max-w-md">
        <h3 class="text-lg font-semibold mb-4">
          {{ purchaseModal.item?.purchased ? 'Edit Purchase' : 'Mark Item as Purchased' }}
        </h3>
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Item</label>
            <div class="p-2 bg-gray-100 rounded">{{ purchaseModal.item?.name }}</div>
          </div>
          <div>
            <label for="actualPrice" class="block text-sm font-medium text-gray-700 mb-1">Actual Price</label>
            <input id="actualPrice" v-model="purchaseModal.actualPrice" type="number"
              class="w-full p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500"
              placeholder="2.99" min="0" step="0.01" />
          </div>
          <div>
            <label for="purchaseStore" class="block text-sm font-medium text-gray-700 mb-1">Store</label>
            <select id="purchaseStore" v-model="purchaseModal.storeId"
              class="w-full p-2 border rounded focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500">
              <option value="">Select a store</option>
              <option v-for="store in stores" :key="store.id" :value="store.id">
                {{ store.name }}
              </option>
            </select>
          </div>
          <div class="flex space-x-3">
            <button @click="closePurchaseModal"
              class="flex-1 bg-gray-200 text-gray-800 py-2 rounded hover:bg-gray-300 transition-colors">
              Cancel
            </button>
            <button @click="markAsPurchased"
              class="flex-1 bg-emerald-600 text-white py-2 rounded hover:bg-emerald-700 transition-colors"
              :disabled="!purchaseModal.actualPrice || !purchaseModal.storeId">
              Confirm
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// Tabs
const tabs = [
  { id: 'lists', name: 'Lists' },
  { id: 'categories', name: 'Categories' },
  { id: 'stores', name: 'Stores' }
]
const activeTab = ref('lists')
const selectedList = ref(null)

// Currency options
const currencies = [
  { code: 'EUR', symbol: '€' },
  { code: 'KES', symbol: 'KES' },
  { code: 'USD', symbol: '$' },
  // { code: 'GBP', symbol: '£' },
  // { code: 'JPY', symbol: '¥' },
  // { code: 'CAD', symbol: 'C$' },
  // { code: 'AUD', symbol: 'A$' },
  // { code: 'INR', symbol: '₹' },
  // { code: 'CNY', symbol: '¥' },
  // { code: 'BRL', symbol: 'R$' },
  // { code: 'MXN', symbol: 'Mex$' }
]
const selectedCurrency = ref(currencies[0])

// Data storage
const categories = ref([])
const stores = ref([])
const shoppingLists = ref([])

// Form inputs
const newCategory = ref('')
const newStore = ref('')
const newList = ref({ name: '', budget: '' })
const newItem = ref({
  name: '',
  categoryId: '',
  expectedPrice: '',
})

// Purchase modal
const purchaseModal = ref({
  show: false,
  item: null,
  actualPrice: '',
  storeId: '',
  isEdit: false
})

// Format currency
const formatCurrency = (amount) => {
  return `${selectedCurrency.value.symbol}${amount.toFixed(2)}`
}

// Load data from localStorage
onMounted(() => {
  const savedCategories = localStorage.getItem('categories')
  const savedStores = localStorage.getItem('stores')
  const savedLists = localStorage.getItem('shoppingLists')
  const savedCurrency = localStorage.getItem('selectedCurrency')

  if (savedCategories) categories.value = JSON.parse(savedCategories)
  if (savedStores) stores.value = JSON.parse(savedStores)
  if (savedLists) shoppingLists.value = JSON.parse(savedLists)
  if (savedCurrency) {
    const currency = JSON.parse(savedCurrency)
    selectedCurrency.value = currencies.find(c => c.code === currency.code) || currencies[0]
  }
})

// Save data to localStorage when it changes
watch(categories, (newVal) => {
  localStorage.setItem('categories', JSON.stringify(newVal))
}, { deep: true })

watch(stores, (newVal) => {
  localStorage.setItem('stores', JSON.stringify(newVal))
}, { deep: true })

watch(shoppingLists, (newVal) => {
  localStorage.setItem('shoppingLists', JSON.stringify(newVal))
}, { deep: true })

watch(selectedCurrency, (newVal) => {
  localStorage.setItem('selectedCurrency', JSON.stringify(newVal))
}, { deep: true })

// Category functions
const addCategory = () => {
  if (!newCategory.value.trim()) return

  categories.value.push({
    id: Date.now().toString(),
    name: newCategory.value.trim()
  })

  newCategory.value = ''
}

const removeCategory = (id) => {
  categories.value = categories.value.filter(category => category.id !== id)
}

const getCategoryName = (id) => {
  const category = categories.value.find(cat => cat.id === id)
  return category ? category.name : 'Unknown'
}

// Store functions
const addStore = () => {
  if (!newStore.value.trim()) return

  stores.value.push({
    id: Date.now().toString(),
    name: newStore.value.trim()
  })

  newStore.value = ''
}

const removeStore = (id) => {
  stores.value = stores.value.filter(store => store.id !== id)
}

const getStoreName = (id) => {
  const store = stores.value.find(s => s.id === id)
  return store ? store.name : 'Unknown'
}

// Shopping list functions
const createList = () => {
  if (!newList.value.name.trim() || !newList.value.budget) return

  shoppingLists.value.push({
    id: Date.now().toString(),
    name: newList.value.name.trim(),
    budget: parseFloat(newList.value.budget),
    items: []
  })

  newList.value = { name: '', budget: '' }
}

const selectList = (list) => {
  selectedList.value = list
  activeTab.value = 'details'
}

const backToLists = () => {
  selectedList.value = null
  activeTab.value = 'lists'
}

const addItemToList = () => {
  if (!newItem.value.name.trim() || !newItem.value.expectedPrice) return

  selectedList.value.items.push({
    id: Date.now().toString(),
    name: newItem.value.name.trim(),
    categoryId: newItem.value.categoryId || null,
    expectedPrice: parseFloat(newItem.value.expectedPrice),
    purchased: false,
    actualPrice: 0,
    storeId: null
  })

  // Reset form
  newItem.value = {
    name: '',
    categoryId: '',
    expectedPrice: '',
  }
}

const calculateRemaining = (list) => {
  if (!list) return 0

  const totalExpected = list.items.reduce((sum, item) => {
    // If item is purchased, use actual price, otherwise use expected price
    return sum + (item.purchased ? item.actualPrice : item.expectedPrice)
  }, 0)

  return list.budget - totalExpected
}

// Purchase modal functions
const openPurchaseModal = (item) => {
  purchaseModal.value = {
    show: true,
    item: item,
    actualPrice: item.purchased ? item.actualPrice.toString() : item.expectedPrice.toString(),
    storeId: item.storeId || '',
    isEdit: item.purchased
  }
}

const closePurchaseModal = () => {
  purchaseModal.value.show = false
}

const markAsPurchased = () => {
  if (!purchaseModal.value.item || !purchaseModal.value.actualPrice || !purchaseModal.value.storeId) return

  const item = selectedList.value.items.find(i => i.id === purchaseModal.value.item.id)
  if (item) {
    item.purchased = true
    item.actualPrice = parseFloat(purchaseModal.value.actualPrice)
    item.storeId = purchaseModal.value.storeId

    // Update expected price for future reference only if it's a new purchase
    if (!purchaseModal.value.isEdit) {
      item.expectedPrice = item.actualPrice
    }
  }

  closePurchaseModal()
}

const unpurchaseItem = (item) => {
  item.purchased = false
  item.actualPrice = 0
  item.storeId = null
}
</script>
