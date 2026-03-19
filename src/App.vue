<template>
  <div class="min-h-screen bg-gray-900 text-gray-100 font-sans p-4 pb-20">
    <header class="max-w-md mx-auto mb-8 pt-4">
      <h1 class="text-3xl font-bold text-orange-500 flex items-center gap-2">
        <WrenchIcon :size="32" /> MotoLog
      </h1>
      <p class="text-gray-400 text-sm">Quản lý bảo dưỡng xe cá nhân</p>
    </header>

    <main class="max-w-md mx-auto space-y-6">
      <div class="bg-gray-800 p-6 rounded-2xl shadow-xl border border-gray-700">
        <label class="block text-sm font-medium text-gray-400 mb-2">Số ODO hiện tại (km)</label>
        <div class="flex gap-2">
          <input 
            v-model.number="currentOdo" 
            type="number" 
            class="flex-1 bg-gray-900 border border-gray-600 rounded-lg px-4 py-3 text-2xl font-mono text-orange-400 focus:outline-none focus:border-orange-500"
            placeholder="0"
          />
          <button @click="saveOdo" class="bg-orange-600 hover:bg-orange-700 px-4 rounded-lg transition-colors">
            Lưu
          </button>
        </div>
      </div>

      <div class="space-y-4">
        <h2 class="text-lg font-semibold flex items-center gap-2">
          <ClipboardListIcon :size="20" /> Tình trạng phụ tùng
        </h2>
        
        <div v-for="(item, index) in maintenanceItems" :key="index" 
          class="bg-gray-800 p-4 rounded-xl border-l-4"
          :class="getStatusColor(item)"
        >
          <div class="flex justify-between items-start">
            <div>
              <h3 class="font-bold text-lg">{{ item.name }}</h3>
              <p class="text-sm text-gray-400">Định kỳ: {{ item.interval }} km</p>
            </div>
            <div class="text-right">
              <span class="text-xs uppercase font-bold px-2 py-1 rounded" :class="getStatusBadge(item)">
                {{ getStatusText(item) }}
              </span>
            </div>
          </div>

          <div class="mt-4 flex justify-between items-end">
            <div>
              <p class="text-xs text-gray-500">Lần cuối: {{ item.lastServiceOdo }} km</p>
              <p class="text-sm font-medium">Còn lại: <span class="text-orange-400">{{ calculateRemaining(item) }} km</span></p>
            </div>
            <button 
              @click="resetService(index)"
              class="bg-gray-700 hover:bg-gray-600 text-xs py-2 px-4 rounded-lg transition-all border border-gray-600"
            >
              Đã thay mới
            </button>
          </div>
        </div>
      </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-gray-800 border-t border-gray-700 p-4 flex justify-around text-gray-400">
      <div class="flex flex-col items-center text-orange-500">
        <HomeIcon :size="24" />
        <span class="text-[10px] mt-1">Trang chủ</span>
      </div>
      <div class="flex flex-col items-center">
        <HistoryIcon :size="24" />
        <span class="text-[10px] mt-1">Lịch sử</span>
      </div>
      <div class="flex flex-col items-center">
        <SettingsIcon :size="24" />
        <span class="text-[10px] mt-1">Cài đặt</span>
      </div>
    </nav>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { 
  WrenchIcon, 
  ClipboardListIcon, 
  HomeIcon, 
  HistoryIcon, 
  SettingsIcon 
} from 'lucide-vue-next'

// 1. Dữ liệu số KM hiện tại
const currentOdo = ref(0)

// 2. Danh sách các hạng mục bảo dưỡng
const maintenanceItems = ref([
  { name: 'Thay nhớt máy', interval: 2000, lastServiceOdo: 0 },
  { name: 'Thay nhớt lap (xe ga)', interval: 6000, lastServiceOdo: 0 },
  { name: 'Vệ sinh nồi/xích', interval: 5000, lastServiceOdo: 0 },
  { name: 'Lọc gió', interval: 10000, lastServiceOdo: 0 },
])

// 3. Các hàm xử lý Logic
const calculateRemaining = (item) => {
  const remaining = (item.lastServiceOdo + item.interval) - currentOdo.value
  return remaining > 0 ? remaining : 0
}

const getStatusColor = (item) => {
  const remaining = calculateRemaining(item)
  if (remaining <= 0) return 'border-red-500 bg-red-900/10'
  if (remaining < 500) return 'border-yellow-500 bg-yellow-900/10'
  return 'border-green-500 bg-green-900/10'
}

const getStatusBadge = (item) => {
  const remaining = calculateRemaining(item)
  if (remaining <= 0) return 'bg-red-500/20 text-red-500'
  if (remaining < 500) return 'bg-yellow-500/20 text-yellow-500'
  return 'bg-green-500/20 text-green-500'
}

const getStatusText = (item) => {
  const remaining = calculateRemaining(item)
  if (remaining <= 0) return 'Cần thay ngay!'
  if (remaining < 500) return 'Sắp đến hạn'
  return 'Ổn định'
}

const resetService = (index) => {
  maintenanceItems.value[index].lastServiceOdo = currentOdo.value
  saveToLocal()
}

// 4. Lưu và Tải dữ liệu từ máy người dùng (LocalStorage)
const saveToLocal = () => {
  localStorage.setItem('motolog_odo', currentOdo.value)
  localStorage.setItem('motolog_items', JSON.stringify(maintenanceItems.value))
}

const saveOdo = () => {
  saveToLocal()
  alert('Đã cập nhật số KM hiện tại!')
}

onMounted(() => {
  const savedOdo = localStorage.getItem('motolog_odo')
  const savedItems = localStorage.getItem('motolog_items')
  
  if (savedOdo) currentOdo.value = parseInt(savedOdo)
  if (savedItems) maintenanceItems.value = JSON.parse(savedItems)
})
</script>

<style>
/* Bạn có thể thêm CSS tùy chỉnh ở đây nếu cần */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
</style>