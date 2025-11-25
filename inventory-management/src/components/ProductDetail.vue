<script setup>
import { ref, computed } from 'vue'

// --- DATA MOCK (Sesuai JSON Kamu) ---
const apiResponse = {
  "message": "success",
  "response": {
    "description": "Keychron K2 Pro adalah keyboard mekanis nirkabel QMK/VIA custom yang sepenuhnya dapat dikustomisasi. Dengan layout 75% yang ringkas, ini adalah pilihan premium untuk Mac dan Windows.",
    "weight": 1.2,
    "pre_order": { "days_to_ship": 2, "is_pre_order": false },
    "item_name": "Keychron K2 Pro Wireless Mechanical Keyboard - RGB Hot-swappable",
    "images": {
      "image_url_list": [
        "https://placehold.co/600x400/222/fff?text=Keychron+Front",
        "https://placehold.co/600x400/333/fff?text=Keychron+Side",
        "https://placehold.co/600x400/444/fff?text=Keychron+Back"
      ]
    },
    "item_status": "NORMAL",
    "price_info": { "current_price": 1850000.00, "original_price": 2100000.00 },
    "logistic_info": [
      { "shipping_fee": 15000.00, "logistic_id": 80001, "is_free": false, "name": "JNE Reguler" },
      { "shipping_fee": 0, "logistic_id": 80002, "is_free": true, "name": "SiCepat Halu" }
    ],
    "item_id": 9988776655,
    "attributes": [
      { "attribute_id": 1001, "attribute_value_list": [{ "original_value_name": "Keychron", "value_unit": "Brand" }] },
      { "attribute_id": 1002, "attribute_value_list": [{ "original_value_name": "Brown Switch", "value_unit": "Switch Type" }] }
    ],
    "condition": "NEW",
    "wholesale": [
      { "min_count": 5, "max_count": 10, "unit_price": 1750000.00 },
      { "min_count": 11, "max_count": 50, "unit_price": 1650000.00 }
    ],
    "brand": { "original_brand_name": "Keychron" },
    "seller_stock": [
      { "location_id": "GUDANG_JKT_01", "stock": 150 },
      { "location_id": "GUDANG_SBY_02", "stock": 25 }
    ]
  }
}

// State
const product = ref(apiResponse.response)
const activeImage = ref(product.value.images.image_url_list[0])

// Helper: Format Rupiah
const formatRupiah = (number) => {
  return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(number)
}

// Helper: Hitung Diskon
const discountPercentage = computed(() => {
  const original = product.value.price_info.original_price
  const current = product.value.price_info.current_price
  if (original > current) {
    return Math.round(((original - current) / original) * 100)
  }
  return 0
})

// Helper: Total Stok
const totalStock = computed(() => {
  return product.value.seller_stock.reduce((acc, curr) => acc + curr.stock, 0)
})

// Fungsi ganti gambar
const setActiveImage = (url) => {
  activeImage.value = url
}
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
      <div class="subtitle">SKU/Item ID: {{ product.item_id }} | Brand: <strong>{{ product.brand.original_brand_name }}</strong></div>
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
            <span>📦 Distribusi Stok</span>
            <span class="total-stock">Total: {{ totalStock }} unit</span>
          </div>
          <table class="simple-table">
            <thead>
              <tr>
                <th>Lokasi Gudang</th>
                <th style="text-align: right;">Jumlah</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="stock in product.seller_stock" :key="stock.location_id">
                <td>{{ stock.location_id }}</td>
                <td style="text-align: right; font-weight: bold;">{{ stock.stock }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="mini-grid">
          <div class="card">
            <div class="card-header">Spesifikasi</div>
            <ul class="attr-list">
              <li v-for="attr in product.attributes" :key="attr.attribute_id">
                <span class="label">{{ attr.attribute_value_list[0].value_unit }}:</span>
                <span class="val">{{ attr.attribute_value_list[0].original_value_name }}</span>
              </li>
              <li>
                <span class="label">Berat:</span>
                <span class="val">{{ product.weight }} Kg</span>
              </li>
            </ul>
          </div>

          <div class="card">
            <div class="card-header">Logistik</div>
            <ul class="logistic-list">
              <li v-for="log in product.logistic_info" :key="log.logistic_id">
                <div class="log-name">ID: {{ log.logistic_id }}</div>
                <div class="log-price">
                  {{ log.is_free ? 'Gratis Ongkir' : formatRupiah(log.shipping_fee) }}
                </div>
              </li>
            </ul>
          </div>
        </div>

        <div class="card" v-if="product.wholesale.length">
          <div class="card-header">Harga Grosir</div>
          <table class="simple-table">
            <thead>
              <tr>
                <th>Min</th>
                <th>Max</th>
                <th>Harga Satuan</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(tier, idx) in product.wholesale" :key="idx">
                <td>{{ tier.min_count }}</td>
                <td>{{ tier.max_count }}</td>
                <td style="color: #d9534f">{{ formatRupiah(tier.unit_price) }}</td>
              </tr>
            </tbody>
          </table>
        </div>

      </div> </div> <div class="card description-card">
      <h3>Deskripsi Produk</h3>
      <p>{{ product.description }}</p>
    </div>

  </div>
</template>

<style scoped>
/* --- BASE LAYOUT --- */
.dashboard-container {
  max-width: 1000px;
  margin: 20px auto;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #333;
}

.header-section { margin-bottom: 20px; }

.content-grid {
  display: grid;
  grid-template-columns: 350px 1fr; /* Kiri Fixed, Kanan Flexible */
  gap: 24px;
}

@media (max-width: 768px) {
  .content-grid { grid-template-columns: 1fr; }
}

/* --- TYPOGRAPHY & BADGES --- */
.product-title {
  font-size: 1.5rem;
  margin: 10px 0 5px 0;
  color: #2c3e50;
}
.subtitle { color: #7f8c8d; font-size: 0.9rem; }

.badge {
  padding: 4px 12px;
  border-radius: 50px;
  font-size: 0.75rem;
  font-weight: bold;
  text-transform: uppercase;
  margin-right: 8px;
}
.status-badge { background: #e1f5fe; color: #0288d1; border: 1px solid #0288d1; }
.condition-badge { background: #e8f5e9; color: #2e7d32; border: 1px solid #2e7d32; }
.po-badge { background: #fff3e0; color: #ef6c00; }

/* --- IMAGES --- */
.image-section { display: flex; flex-direction: column; gap: 10px; }
.main-image-frame {
  width: 100%;
  height: 350px;
  border: 1px solid #eee;
  border-radius: 8px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background: white;
}
.main-image { max-width: 100%; max-height: 100%; object-fit: contain; }

.thumbnail-list { display: flex; gap: 8px; overflow-x: auto; }
.thumb-frame {
  width: 60px;
  height: 60px;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  overflow: hidden;
  opacity: 0.6;
  transition: all 0.2s;
}
.thumb-frame.active { border: 2px solid #0078d4; opacity: 1; }
.thumb-frame img { width: 100%; height: 100%; object-fit: cover; }

/* --- CARDS & INFO --- */
.card {
  background: white;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 16px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.03);
}

.card-header {
  font-weight: 600;
  border-bottom: 1px solid #f0f0f0;
  padding-bottom: 8px;
  margin-bottom: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #444;
}

/* --- PRICE --- */
.price-card { background: #fdfdfd; border-left: 4px solid #0078d4; }
.price-header { font-size: 0.85rem; color: #666; margin-bottom: 4px; }
.current-price { font-size: 1.8rem; font-weight: bold; color: #d0011b; } /* Warna Shopee Price */
.price-row { display: flex; align-items: baseline; gap: 12px; }
.original-price { text-decoration: line-through; color: #999; font-size: 0.9rem; }
.discount-badge { background: #ffebee; color: #d0011b; padding: 2px 6px; border-radius: 4px; font-size: 0.8rem; font-weight: bold; }

/* --- TABLES --- */
.simple-table { width: 100%; border-collapse: collapse; font-size: 0.9rem; }
.simple-table th { text-align: left; color: #666; padding: 8px 4px; border-bottom: 1px solid #eee; font-weight: 600; }
.simple-table td { padding: 8px 4px; border-bottom: 1px solid #f9f9f9; }
.total-stock { font-size: 0.85rem; background: #eee; padding: 2px 8px; border-radius: 4px; }

/* --- MINI GRID & LISTS --- */
.mini-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
.attr-list, .logistic-list { list-style: none; padding: 0; margin: 0; font-size: 0.9rem; }
.attr-list li { display: flex; justify-content: space-between; padding: 4px 0; border-bottom: 1px dashed #eee; }
.attr-list .label { color: #666; }
.logistic-list li { display: flex; justify-content: space-between; padding: 4px 0; }
.log-price { font-weight: bold; color: #2e7d32; }

/* --- DESCRIPTION --- */
.description-card h3 { margin-top: 0; font-size: 1.1rem; }
.description-card p { line-height: 1.6; color: #555; }
</style>