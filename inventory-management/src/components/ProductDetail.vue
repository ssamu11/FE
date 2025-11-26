<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  initialData: {
    type: Object,
    required: true
  }
})

// --- LOGIKA UTAMA: NORMALISASI DATA (COMPLEX vs SIMPLE) ---
const product = computed(() => {
  // Ambil data mentah (bisa dari API Create yang simple, atau API List yang kompleks)
  const data = props.initialData || {}; 
  
  // 1. Normalisasi Harga
  // Cek 'price' (format simple) atau 'price_info.current_price' (format kompleks)
  let currentPrice = 0;
  if (typeof data.price !== 'undefined') {
    currentPrice = parseInt(data.price);
  } else if (data.price_info && data.price_info.current_price) {
    currentPrice = parseInt(data.price_info.current_price);
  }

  // Cek Original Price (Harga Coret)
  // HAPUS LOGIKA * 1.1 agar barang inputan manual tidak otomatis terlihat diskon
  let originalPrice = currentPrice; 
  if (data.price_info && data.price_info.original_price) {
    originalPrice = parseInt(data.price_info.original_price);
  }

  // 2. Normalisasi Stok
  // API Create pakai 'init_stock', API List pakai 'seller_stock' array
  let stockList = [];
  if (data.seller_stock && Array.isArray(data.seller_stock)) {
    stockList = data.seller_stock;
  } else {
    // Jika format simple, buat array dummy agar tabel stok tetap muncul cantik
    const singleStock = data.init_stock || data.stock || 0;
    stockList = [{ location_id: "GUDANG_PUSAT", stock: parseInt(singleStock) }];
  }

  // 3. Normalisasi Gambar
  let imgList = [];
  if (data.images && data.images.image_url_list && data.images.image_url_list.length > 0) {
    imgList = data.images.image_url_list;
  } else {
    // Placeholder Generator
    const nameEnc = encodeURIComponent(data.name || data.item_name || 'Product');
    imgList = [
      `https://placehold.co/600x400/0078d4/fff?text=${nameEnc}`,
      `https://placehold.co/600x400/555/fff?text=Side+View`,
    ];
  }

  // Return Object Standar UI
  return {
    ...data, // Spread data asli untuk backup
    item_name: data.name || data.item_name || 'Tanpa Nama',
    item_id: data.id || data.item_id || 'LOCAL-ID', // Handle ID
    item_status: data.item_status || 'ACTIVE',
    condition: data.condition || 'New',
    pre_order: data.pre_order || { is_pre_order: false },
    brand: data.brand || { original_brand_name: "General" }, // Default brand
    description: data.description || "Deskripsi produk belum ditambahkan.",
    weight: data.weight || 1,
    
    // Field yang sudah dinormalisasi
    price_info: {
      current_price: currentPrice,
      original_price: originalPrice
    },
    images: {
      image_url_list: imgList
    },
    seller_stock: stockList,
    // Pastikan array kosong jika data tidak ada (agar tidak error v-for)
    attributes: data.attributes || [],
    logistic_info: data.logistic_info || [],
    wholesale: data.wholesale || []
  }
})

// State untuk gambar aktif
const activeImage = ref('')

// Watcher agar gambar mereset saat ganti produk
watch(product, (newVal) => {
  if (newVal && newVal.images.image_url_list.length > 0) {
    activeImage.value = newVal.images.image_url_list[0]
  }
}, { immediate: true })

// --- HELPER FORMATTER ---
const formatRupiah = (number) => {
  return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number || 0)
}

const discountPercentage = computed(() => {
  const original = product.value.price_info.original_price
  const current = product.value.price_info.current_price
  // Hitung diskon hanya jika harga asli lebih besar dari harga sekarang
  if (original > current && original > 0) {
    return Math.round(((original - current) / original) * 100)
  }
  return 0
})

const totalStock = computed(() => {
  if(!product.value.seller_stock) return 0;
  return product.value.seller_stock.reduce((acc, curr) => acc + parseInt(curr.stock), 0)
})

const setActiveImage = (url) => { activeImage.value = url }
</script>

<template>
  <div class="dashboard-container">
    
    <div class="header-section">
      <div class="badges">
        <span class="badge status-badge">{{ product.item_status }}</span>
        <span class="badge condition-badge">{{ product.condition }}</span>
        <span v-if="product.pre_order.is_pre_order" class="badge po-badge">Pre-Order</span>
      </div>
      <h1 class="product-title">{{ product.item_name }}</h1>
      <div class="subtitle">
        ID: {{ product.item_id }} | Brand: <strong>{{ product.brand.original_brand_name }}</strong>
      </div>
    </div>

    <div class="content-grid">
      
      <div class="image-section">
        <div class="main-image-frame">
          <img :src="activeImage" alt="Main Product" class="main-image" />
        </div>
        <div class="thumbnail-list">
          <div 
            v-for="(img, index) in product.images.image_url_list" 
            :key="index"
            class="thumb-frame"
            :class="{ active: activeImage === img }"
            @click="setActiveImage(img)"
          >
            <img :src="img" alt="thumb" />
          </div>
        </div>
      </div>

      <div class="info-section">
        
        <div class="card price-card">
          <div class="price-header">Harga Jual</div>
          <div class="price-row">
            <span class="current-price">{{ formatRupiah(product.price_info.current_price) }}</span>
            
            <div v-if="discountPercentage > 0" class="discount-block">
              <span class="original-price">{{ formatRupiah(product.price_info.original_price) }}</span>
              <span class="discount-badge">{{ discountPercentage }}% OFF</span>
            </div>
          </div>
        </div>

        <div class="card">
          <div class="card-header">
            <span>📦 Informasi Stok</span>
            <span class="total-stock">Total: {{ totalStock }}</span>
          </div>
          <table class="simple-table">
            <thead>
              <tr>
                <th>Lokasi</th>
                <th style="text-align: right;">Jumlah</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(stock, idx) in product.seller_stock" :key="idx">
                <td>{{ stock.location_id }}</td>
                <td style="text-align: right; font-weight: bold;">{{ stock.stock }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="mini-grid">
          <div class="card" v-if="product.attributes && product.attributes.length > 0">
            <div class="card-header">Spesifikasi</div>
            <ul class="attr-list">
              <li v-for="attr in product.attributes" :key="attr.attribute_id">
                <span class="label">{{ attr.attribute_value_list[0].value_unit || 'Spec' }}:</span>
                <span class="val">{{ attr.attribute_value_list[0].original_value_name }}</span>
              </li>
            </ul>
          </div>
          
          <div class="card" v-if="product.logistic_info && product.logistic_info.length > 0">
            <div class="card-header">Logistik</div>
            <ul class="logistic-list">
              <li v-for="log in product.logistic_info" :key="log.logistic_id">
                <div class="log-name">{{ log.logistic_name || 'Kurir' }}</div>
                <div class="log-price">
                  {{ log.is_free ? 'Gratis' : formatRupiah(log.shipping_fee) }}
                </div>
              </li>
            </ul>
          </div>
        </div>

        <div class="card" v-if="product.wholesale && product.wholesale.length > 0">
          <div class="card-header">Harga Grosir</div>
          <table class="simple-table">
            <thead>
              <tr>
                <th>Min</th>
                <th>Max</th>
                <th>Harga</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(tier, idx) in product.wholesale" :key="idx">
                <td>{{ tier.min_count }}</td>
                <td>{{ tier.max_count }}</td>
                <td style="color: #d0011b">{{ formatRupiah(tier.unit_price) }}</td>
              </tr>
            </tbody>
          </table>
        </div>

      </div> 
    </div> 

    <div class="card description-card">
      <h3>Deskripsi Produk</h3>
      <p style="white-space: pre-line;">{{ product.description }}</p>
    </div>

  </div>
</template>

<style scoped>
/* STYLE UTAMA - Tidak banyak berubah, hanya perapihan */
.dashboard-container { max-width: 1000px; margin: 20px auto; font-family: 'Segoe UI', sans-serif; color: #333; }
.header-section { margin-bottom: 20px; }
.content-grid { display: grid; grid-template-columns: 350px 1fr; gap: 24px; }
@media (max-width: 768px) { .content-grid { grid-template-columns: 1fr; } }

.product-title { font-size: 1.5rem; margin: 10px 0 5px 0; color: #2c3e50; }
.subtitle { color: #7f8c8d; font-size: 0.9rem; }
.badge { padding: 4px 12px; border-radius: 50px; font-size: 0.75rem; font-weight: bold; text-transform: uppercase; margin-right: 8px; }
.status-badge { background: #e1f5fe; color: #0288d1; border: 1px solid #0288d1; }
.condition-badge { background: #e8f5e9; color: #2e7d32; border: 1px solid #2e7d32; }
.po-badge { background: #fff3e0; color: #ef6c00; }

.image-section { display: flex; flex-direction: column; gap: 10px; }
.main-image-frame { width: 100%; height: 350px; border: 1px solid #eee; border-radius: 8px; overflow: hidden; display: flex; align-items: center; justify-content: center; background: white; }
.main-image { max-width: 100%; max-height: 100%; object-fit: contain; }
.thumbnail-list { display: flex; gap: 8px; overflow-x: auto; }
.thumb-frame { width: 60px; height: 60px; border: 1px solid #ddd; border-radius: 6px; cursor: pointer; overflow: hidden; opacity: 0.6; transition: all 0.2s; }
.thumb-frame.active { border: 2px solid #0078d4; opacity: 1; }
.thumb-frame img { width: 100%; height: 100%; object-fit: cover; }

.card { background: white; border: 1px solid #e0e0e0; border-radius: 8px; padding: 16px; margin-bottom: 16px; box-shadow: 0 2px 4px rgba(0,0,0,0.03); }
.card-header { font-weight: 600; border-bottom: 1px solid #f0f0f0; padding-bottom: 8px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; color: #444; }

.price-card { background: #fdfdfd; border-left: 4px solid #0078d4; }
.price-header { font-size: 0.85rem; color: #666; margin-bottom: 4px; }
.current-price { font-size: 1.8rem; font-weight: bold; color: #d0011b; }
.price-row { display: flex; align-items: baseline; gap: 12px; }
.original-price { text-decoration: line-through; color: #999; font-size: 0.9rem; }
.discount-badge { background: #ffebee; color: #d0011b; padding: 2px 6px; border-radius: 4px; font-size: 0.8rem; font-weight: bold; }
.discount-block { display: flex; align-items: center; gap: 8px; }

.simple-table { width: 100%; border-collapse: collapse; font-size: 0.9rem; }
.simple-table th { text-align: left; color: #666; padding: 8px 4px; border-bottom: 1px solid #eee; font-weight: 600; }
.simple-table td { padding: 8px 4px; border-bottom: 1px solid #f9f9f9; }
.total-stock { font-size: 0.85rem; background: #eee; padding: 2px 8px; border-radius: 4px; }

.mini-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
.attr-list, .logistic-list { list-style: none; padding: 0; margin: 0; font-size: 0.9rem; }
.attr-list li { display: flex; justify-content: space-between; padding: 4px 0; border-bottom: 1px dashed #eee; }
.attr-list .label { color: #666; }
.logistic-list li { display: flex; justify-content: space-between; padding: 4px 0; }
.log-price { font-weight: bold; color: #2e7d32; }

.description-card h3 { margin-top: 0; font-size: 1.1rem; }
.description-card p { line-height: 1.6; color: #555; }
</style>