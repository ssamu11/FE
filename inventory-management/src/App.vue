<script setup>
import { ref, onMounted } from 'vue'
import ProductDetail from './components/ProductDetail.vue'

// --- PENGATURAN MODE ---
const USE_MOCK_DATA = true 

// State
const products = ref([])
const isLoading = ref(false)
const errorMessage = ref('')
const form = ref({ sku: '', name: '', stock: 0, price: 0 })

// State untuk Navigasi (List vs Detail)
const selectedProduct = ref(null)

// --- DATA MOCK LIST ---
const mockDatabase = ref([
  { id: 'sku-001', name: 'Laptop Gaming ROG', stock: 5, price: 15000000 },
  { id: 'sku-002', name: 'Mouse Wireless Logitech', stock: 20, price: 150000 },
  { id: 'sku-003', name: 'Monitor 24 Inch Samsung', stock: 10, price: 2500000 }
])

// --- FUNGSI UTAMA ---

async function fetchProducts() {
  isLoading.value = true
  try {
    if (USE_MOCK_DATA) {
      // Simulasi loading
      await new Promise(r => setTimeout(r, 500)) 
      products.value = [...mockDatabase.value]
    } else {
      const response = await fetch('/api/GetProducts')
      if (!response.ok) throw new Error('Gagal ambil data')
      const data = await response.json()
      products.value = data.products || data
    }
  } catch (error) {
    errorMessage.value = error.message
  } finally {
    isLoading.value = false
  }
}

async function addProduct() {
  if (!form.value.sku || !form.value.name) return alert("Wajib diisi!")

  const newProduct = {
    id: form.value.sku,
    name: form.value.name,
    stock: parseInt(form.value.stock),
    price: parseInt(form.value.price)
  }

  isLoading.value = true
  try {
    if (USE_MOCK_DATA) {
      await new Promise(r => setTimeout(r, 500))
      mockDatabase.value.push(newProduct)
      products.value = [...mockDatabase.value]
      alert("Sukses tambah data (Mock)")
    } else {
      await fetch('/api/AddProduct', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newProduct)
      })
      await fetchProducts()
    }
    form.value = { sku: '', name: '', stock: 0, price: 0 }
  } catch (e) {
    alert(e.message)
  } finally {
    isLoading.value = false
  }
}

// --- LOGIKA NAVIGASI DETAIL ---

function viewDetail(simpleItem) {
  // Disini MAGIC-nya: Kita ubah data simple tabel jadi data kompleks Shopee
  // Agar UI Detail yang bagus itu bisa merender datanya.
  
  const complexData = {
    message: "success",
    response: {
      item_name: simpleItem.name,
      item_id: simpleItem.id,
      item_status: "NORMAL",
      condition: "NEW",
      description: `Ini adalah deskripsi detail untuk produk ${simpleItem.name}. Produk ini memiliki kualitas premium dan stok tersedia di gudang kami.`,
      brand: { original_brand_name: "Generic Brand" },
      weight: 1.5,
      pre_order: { is_pre_order: false },
      price_info: {
        current_price: simpleItem.price,
        original_price: simpleItem.price * 1.2 // Simulasi harga coret
      },
      // Stok kita pecah pura-pura jadi 2 gudang
      seller_stock: [
        { location_id: "GUDANG_JKT", stock: Math.floor(simpleItem.stock / 2) },
        { location_id: "GUDANG_SBY", stock: Math.ceil(simpleItem.stock / 2) }
      ],
      images: {
        image_url_list: [
          "https://placehold.co/600x400/222/fff?text=" + encodeURIComponent(simpleItem.name),
          "https://placehold.co/600x400/555/fff?text=Side+View",
          "https://placehold.co/600x400/888/fff?text=Back+View"
        ]
      },
      attributes: [
        { attribute_id: 1, attribute_value_list: [{ value_unit: "Garansi", original_value_name: "1 Tahun" }] }
      ],
      logistic_info: [
        { logistic_id: 1, name: "JNE", shipping_fee: 15000, is_free: false },
        { logistic_id: 2, name: "SiCepat", shipping_fee: 0, is_free: true }
      ],
      wholesale: []
    }
  }

  // Set data yang sudah dikonversi ke state, dan UI akan berubah otomatis
  selectedProduct.value = complexData.response
}

function closeDetail() {
  selectedProduct.value = null
}

onMounted(() => {
  fetchProducts()
})
</script>

<template>
  <div class="container">
    
    <div class="header-nav">
      <h1>📦 Inventory System</h1>
      <button v-if="selectedProduct" @click="closeDetail" class="btn-back">
        ⬅ Kembali ke List
      </button>
    </div>

    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>

    <div v-if="!selectedProduct">
      
      <div v-if="USE_MOCK_DATA" class="alert-mock">
        ⚠️ Mode Data Palsu (Mock) Aktif
      </div>

      <div class="card">
        <h3>Tambah Barang</h3>
        <div class="form-group">
          <input v-model="form.sku" placeholder="SKU" />
          <input v-model="form.name" placeholder="Nama Barang" />
          <input type="number" v-model="form.stock" placeholder="Stok" />
          <input type="number" v-model="form.price" placeholder="Harga" />
          <button @click="addProduct" :disabled="isLoading">Simpan</button>
        </div>
      </div>

      <div class="card">
        <h3>Daftar Stok</h3>
        <table class="inventory-table">
          <thead>
            <tr>
              <th>SKU</th>
              <th>Nama Barang</th>
              <th>Stok</th>
              <th>Harga</th>
              <th>Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in products" :key="item.id">
              <td>{{ item.id }}</td>
              <td>{{ item.name }}</td>
              <td>{{ item.stock }}</td>
              <td>Rp {{ item.price.toLocaleString() }}</td>
              <td>
                <button class="btn-detail" @click="viewDetail(item)">
                  🔍 Detail
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-else>
      <ProductDetail :initialData="selectedProduct" />
    </div>

  </div>
</template>

<style scoped>
.container { max-width: 1000px; margin: 0 auto; font-family: 'Segoe UI', sans-serif; padding: 20px; }
.header-nav { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.card { border: 1px solid #e0e0e0; padding: 20px; margin-bottom: 20px; border-radius: 8px; background: white; }
.form-group { display: flex; gap: 10px; }
input { padding: 10px; border: 1px solid #ccc; border-radius: 4px; flex: 1; }
button { padding: 10px 20px; background-color: #0078d4; color: white; border: none; border-radius: 4px; cursor: pointer; }
.btn-back { background-color: #555; }
.btn-detail { background-color: #fff; border: 1px solid #0078d4; color: #0078d4; padding: 5px 10px; font-size: 0.9em; }
.btn-detail:hover { background-color: #e3f2fd; }
.inventory-table { width: 100%; border-collapse: collapse; }
.inventory-table th, .inventory-table td { padding: 12px; border-bottom: 1px solid #eee; text-align: left; }
.alert-mock { background-color: #fff3cd; color: #856404; padding: 10px; margin-bottom: 15px; border-radius: 4px; text-align: center; }
</style>