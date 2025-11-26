<script setup>
import { ref, onMounted } from 'vue'
import ProductDetail from './components/ProductDetail.vue'

// --- 1. KONFIGURASI URL BACKEND LIVE ---
// Base URL mengarah ke /api, nanti ditambah endpoint spesifik
const API_BASE_URL = 'https://product-service-k3.azurewebsites.net/api'

// --- STATE MANAGEMENT ---
const products = ref([])
const isLoading = ref(false)
const errorMessage = ref('')
const selectedProduct = ref(null)

// FORM STATE: Disesuaikan dengan kebutuhan API (Name, Stock, Price)
const form = ref({ 
  name: '', 
  stock: 0, 
  price: 0 
})

// --- 2. FUNGSI NORMALISASI DATA (READ) ---
function normalizeProduct(item) {
  let finalPrice = item.price;
  if (!finalPrice && item.price_info) finalPrice = item.price_info.current_price;

  let finalStock = 0;
  if (item.init_stock) {
    finalStock = parseInt(item.init_stock);
  } else if (item.seller_stock && Array.isArray(item.seller_stock)) {
    finalStock = item.seller_stock.reduce((acc, curr) => acc + curr.stock, 0);
  } else if (item.stock) {
    finalStock = parseInt(item.stock);
  }

  return {
    id: item.id || item.item_id, 
    name: item.name || item.item_name || 'Produk Tanpa Nama',
    price: parseInt(finalPrice || 0),
    stock: finalStock,
    originalData: item 
  }
}

// --- 3. FETCH DATA (READ) ---
async function fetchProducts() {
  isLoading.value = true
  errorMessage.value = ''
  try {
    const response = await fetch(`${API_BASE_URL}/product/products`)
    if (!response.ok) throw new Error('Gagal mengambil data dari server')
    const rawData = await response.json()
    products.value = rawData.map(normalizeProduct)
  } catch (error) {
    console.error(error)
    errorMessage.value = "Gagal memuat data. Pastikan Backend Azure aktif!"
  } finally {
    isLoading.value = false
  }
}

// --- 4. TAMBAH DATA (CREATE) - DIPERBAIKI ---
async function addProduct() {
  // Validasi sederhana
  if (!form.value.name) return alert("Nama barang wajib diisi!")
  if (form.value.price <= 0) return alert("Harga harus lebih dari 0!")

  // KONSTRUKSI PAYLOAD SESUAI JSON CONTOH
  // { "name": "...", "init_stock": 5, "price": 350000 }
  const payload = {
    name: form.value.name,
    init_stock: parseInt(form.value.stock), // Backend minta 'init_stock'
    price: parseInt(form.value.price)
  }

  isLoading.value = true
  try {
    // Endpoint: https://product-service-k3.azurewebsites.net/api/product/create
    const response = await fetch(`${API_BASE_URL}/product/create`, {
      method: 'POST',
      headers: { 
        'Content-Type': 'application/json' 
      },
      body: JSON.stringify(payload)
    })

    if (!response.ok) {
        // Coba baca pesan error dari server jika ada
        const errorData = await response.text(); 
        throw new Error(errorData || 'Gagal menyimpan data ke Azure')
    }

    // Jika sukses
    alert("Berhasil create data di Azure!")
    await fetchProducts() // Refresh tabel agar data baru muncul
    
    // Reset form
    form.value = { name: '', stock: 0, price: 0 } 

  } catch (error) {
    console.error(error)
    alert("Error: " + error.message)
  } finally {
    isLoading.value = false
  }
}

// --- NAVIGASI ---
function viewDetail(item) {
  selectedProduct.value = item.originalData
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
      <h1>📦 Inventory System (Live Azure)</h1>
      <button v-if="selectedProduct" @click="closeDetail" class="btn-back">
        ⬅ Kembali ke List
      </button>
    </div>

    <div v-if="errorMessage" class="error">{{ errorMessage }}</div>

    <div v-if="!selectedProduct">
      
      <div class="card">
        <h3>Tambah Barang Baru (API Create)</h3>
        <div class="form-group">
          <input 
            v-model="form.name" 
            placeholder="Nama Barang (ex: Inno X3)" 
            type="text"
          />
          
          <input 
            type="number" 
            v-model="form.stock" 
            placeholder="Stok Awal" 
            min="0"
          />
          
          <input 
            type="number" 
            v-model="form.price" 
            placeholder="Harga (Rp)" 
            min="0"
          />
          
          <button @click="addProduct" :disabled="isLoading">
            {{ isLoading ? 'Menyimpan...' : 'Simpan ke Azure' }}
          </button>
        </div>
        <small style="color: #666; font-size: 0.8em; margin-top: 5px; display:block;">
            *Data akan dikirim ke endpoint: <b>/api/product/create</b>
        </small>
      </div>

      <div class="card">
        <h3>Daftar Stok</h3>
        <p v-if="isLoading" class="loading-text">Sedang sinkronisasi dengan Azure...</p>
        
        <table v-else class="inventory-table">
          <thead>
            <tr>
              <th>Nama Barang</th>
              <th>Stok</th>
              <th>Harga</th>
              <th>Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in products" :key="item.id">
              <td>{{ item.name }}</td>
              <td>
                <span :class="{'stock-low': item.stock < 5, 'stock-ok': item.stock >= 5}">
                  {{ item.stock }}
                </span>
              </td>
              <td>Rp {{ item.price.toLocaleString('id-ID') }}</td>
              <td>
                <button class="btn-detail" @click="viewDetail(item)">
                  🔍 Detail
                </button>
              </td>
            </tr>
            <tr v-if="products.length === 0">
              <td colspan="4" style="text-align: center;">Tidak ada data ditemukan.</td>
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
/* CSS SAMA SEPERTI SEBELUMNYA */
.container { max-width: 1000px; margin: 0 auto; font-family: 'Segoe UI', sans-serif; padding: 20px; }
.header-nav { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
.card { border: 1px solid #e0e0e0; padding: 20px; margin-bottom: 20px; border-radius: 8px; background: white; box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
.form-group { display: flex; gap: 10px; flex-wrap: wrap; }
input { padding: 10px; border: 1px solid #ccc; border-radius: 4px; flex: 1; min-width: 150px; }
button { padding: 10px 20px; background-color: #0078d4; color: white; border: none; border-radius: 4px; cursor: pointer; transition: background 0.2s; }
button:hover { background-color: #005a9e; }
button:disabled { background-color: #ccc; cursor: not-allowed; }
.btn-back { background-color: #555; }
.btn-detail { background: none; border: 1px solid #0078d4; color: #0078d4; padding: 6px 12px; font-size: 0.9em; }
.btn-detail:hover { background-color: #e3f2fd; }
.inventory-table { width: 100%; border-collapse: collapse; margin-top: 10px; }
.inventory-table th, .inventory-table td { padding: 12px; border-bottom: 1px solid #eee; text-align: left; }
.inventory-table th { background-color: #f8f9fa; color: #444; font-weight: 600; }
.error { background: #ffebee; color: #c62828; padding: 15px; border-radius: 4px; margin-bottom: 20px; border: 1px solid #ffcdd2; }
.loading-text { color: #0078d4; font-weight: 500; text-align: center; padding: 20px; }
.stock-low { color: #d32f2f; font-weight: bold; }
.stock-ok { color: #2e7d32; font-weight: bold; }
</style>