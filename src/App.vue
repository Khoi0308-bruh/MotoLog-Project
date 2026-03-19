<template>
  <div class="min-h-screen bg-gray-950 text-gray-100 font-sans pb-32">
    <header class="bg-gray-900 border-b border-gray-800 p-6 sticky top-0 z-10 shadow-lg">
      <div class="max-w-md mx-auto flex justify-between items-center">
        <div>
          <h1 class="text-2xl font-black text-orange-500 flex items-center gap-2 italic">
            <WrenchIcon :size="28" /> MOTOLOG PRO
          </h1>
          <p class="text-[10px] text-gray-500 tracking-widest uppercase mt-1">Hệ thống quản lý xe cá nhân</p>
        </div>
        <div class="text-right">
          <p class="text-[10px] text-gray-400 uppercase">Tổng chi phí</p>
          <p class="text-lg font-mono font-bold text-green-400">{{ formatMoney(totalSpent) }}</p>
        </div>
      </div>
    </header>

    <main class="max-w-md mx-auto p-4 mt-4 space-y-6">
      <div v-if="activeTab === 'home'" class="space-y-6">
        
        <div class="bg-gray-900 p-5 rounded-2xl border border-gray-800 shadow-2xl">
          <label class="text-[10px] font-bold text-gray-500 uppercase mb-3 block">Số ODO hiện tại (km)</label>
          <div class="flex items-stretch bg-gray-950 border border-gray-700 rounded-xl overflow-hidden focus-within:border-orange-500 transition-all">
            <input v-model.number="currentOdo" type="number" 
              class="flex-1 bg-transparent px-4 py-3 text-2xl font-mono text-orange-400 outline-none w-full" />
            <button @click="saveData" class="bg-orange-600 hover:bg-orange-500 px-6 font-black text-xs transition-all active:scale-95 border-l border-gray-700">
              LƯU
            </button>
          </div>
        </div>

        <div class="space-y-4">
          <div class="flex justify-between items-center px-1">
            <h2 class="text-xs font-bold text-gray-400 uppercase tracking-wider">Tình trạng phụ tùng</h2>
            <button @click="showAddModal = true" class="text-orange-500 text-[10px] font-black hover:underline">+ THÊM MÓN MỚI</button>
          </div>
          
          <div v-for="(item, index) in maintenanceItems" :key="index" 
            class="bg-gray-900 p-5 rounded-2xl border border-gray-800 relative group overflow-hidden">
            <div class="absolute left-0 top-0 bottom-0 w-1" :class="getHealthColor(item)"></div>

            <div class="flex justify-between items-start mb-4">
              <div>
                <h3 class="font-bold text-lg leading-none">{{ item.name }}</h3>
                <p class="text-[10px] text-gray-500 mt-1">Định kỳ: {{ item.interval }} km</p>
              </div>
              <button @click="deleteItem(index)" class="text-gray-700 hover:text-red-500 transition-colors">
                <TrashIcon :size="18" />
              </button>
            </div>

            <div class="space-y-2">
              <div class="w-full bg-gray-950 h-2 rounded-full overflow-hidden">
                <div class="h-full transition-all duration-700 ease-out" 
                  :style="{ width: getHealth(item) + '%' }"
                  :class="getHealthColor(item)">
                </div>
              </div>
              <div class="flex justify-between items-end">
                <div class="text-[10px]">
                  <span class="text-gray-500 uppercase font-bold">Còn lại:</span>
                  <span class="ml-1 font-mono font-bold text-sm" :class="getHealthTextColor(item)">{{ calculateRemaining(item) }} km</span>
                </div>
                <button @click="openServiceModal(index)" 
                  class="bg-gray-800 hover:bg-orange-600/10 hover:text-orange-500 hover:border-orange-500/50 text-[10px] font-black py-2 px-5 rounded-lg border border-gray-700 transition-all uppercase tracking-tighter">
                  Bảo trì
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div v-if="activeTab === 'history'" class="space-y-4">
        <h2 class="text-xl font-bold mb-6 italic text-orange-500">NHẬT KÝ BẢO TRÌ</h2>
         </div>
    </main>

    <nav class="fixed bottom-6 left-1/2 -translate-x-1/2 bg-gray-900/80 backdrop-blur-xl border border-gray-700/50 px-8 py-3 rounded-2xl flex gap-16 shadow-[0_20px_50px_rgba(0,0,0,0.5)] z-50">
      <button @click="activeTab = 'home'" :class="activeTab === 'home' ? 'text-orange-500 scale-125' : 'text-gray-600'" class="transition-all duration-300">
        <HomeIcon :size="24" :stroke-width="activeTab === 'home' ? 3 : 2" />
      </button>
      <button @click="activeTab = 'history'" :class="activeTab === 'history' ? 'text-orange-500 scale-125' : 'text-gray-600'" class="transition-all duration-300">
        <HistoryIcon :size="24" :stroke-width="activeTab === 'history' ? 3 : 2" />
      </button>
    </nav>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { WrenchIcon, HomeIcon, HistoryIcon, TrashIcon } from 'lucide-vue-next'

// --- DỮ LIỆU ---
const activeTab = ref('home')
const currentOdo = ref(0)
const showAddModal = ref(false)
const showServiceModal = ref(false)
const selectedItemIndex = ref(null)

const maintenanceItems = ref([
  { name: 'Thay nhớt máy', interval: 2000, lastServiceOdo: 0 },
  { name: 'Thay nhớt lap', interval: 6000, lastServiceOdo: 0 },
])

const serviceLogs = ref([])

const newItem = ref({ name: '', interval: null })
const serviceForm = ref({ price: 0, note: '' })

// --- LOGIC TÍNH TOÁN ---
const calculateRemaining = (item) => {
  const rem = (item.lastServiceOdo + item.interval) - currentOdo.value
  return rem > 0 ? rem : 0
}

const getHealth = (item) => {
  const rem = calculateRemaining(item)
  const percent = (rem / item.interval) * 100
  return Math.max(0, Math.min(100, percent))
}

const getHealthColor = (item) => {
  const h = getHealth(item)
  if (h <= 0) return 'bg-red-600'
  if (h < 25) return 'bg-yellow-500'
  return 'bg-green-500'
}

const getHealthTextColor = (item) => {
  const h = getHealth(item)
  if (h <= 0) return 'text-red-500'
  if (h < 25) return 'text-yellow-500'
  return 'text-green-500'
}

const totalSpent = computed(() => {
  return serviceLogs.value.reduce((sum, log) => sum + (log.price || 0), 0)
})

const sortedLogs = computed(() => {
  return [...serviceLogs.value].reverse()
})

const formatMoney = (val) => {
  return new Intl.NumberFormat('vi-VN', { style: 'currency', currency: 'VND' }).format(val)
}

// --- HÀNH ĐỘNG ---
const addNewItem = () => {
  if (newItem.value.name && newItem.value.interval) {
    maintenanceItems.value.push({ ...newItem.value, lastServiceOdo: currentOdo.value })
    newItem.value = { name: '', interval: null }
    showAddModal.value = false
    saveData()
  }
}

const deleteItem = (index) => {
  if (confirm('Bạn có chắc muốn xóa hạng mục này?')) {
    maintenanceItems.value.splice(index, 1)
    saveData()
  }
}

const openServiceModal = (index) => {
  selectedItemIndex.value = index
  serviceForm.value = { price: 0, note: '' }
  showServiceModal.value = true
}

const confirmService = () => {
  const item = maintenanceItems.value[selectedItemIndex.value]
  
  // Lưu vào lịch sử
  serviceLogs.value.push({
    itemName: item.name,
    odo: currentOdo.value,
    price: serviceForm.value.price,
    note: serviceForm.value.note,
    date: new Date().toLocaleDateString('vi-VN')
  })

  // Reset trạng thái phụ tùng
  item.lastServiceOdo = currentOdo.value
  showServiceModal.value = false
  saveData()
}

const saveData = () => {
  localStorage.setItem('motolog_odo', currentOdo.value)
  localStorage.setItem('motolog_items', JSON.stringify(maintenanceItems.value))
  localStorage.setItem('motolog_logs', JSON.stringify(serviceLogs.value))
}

onMounted(() => {
  const o = localStorage.getItem('motolog_odo')
  const i = localStorage.getItem('motolog_items')
  const l = localStorage.getItem('motolog_logs')
  if (o) currentOdo.value = parseInt(o)
  if (i) maintenanceItems.value = JSON.parse(i)
  if (l) serviceLogs.value = JSON.parse(l)
})
</script>