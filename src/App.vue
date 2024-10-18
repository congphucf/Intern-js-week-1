<script setup>
import { ref } from 'vue'
import './assets/main.css'
import axios from 'axios';

const stores = ref([])
const products = ref([])
const storeProduct = ref([])
const choosedProduct = ref([])
const selected = ref('Name (ACS)')
const choosedStore = ref(-1)

function handleChangeStore(id) {
  choosedStore.value = id
  const choosedProductID = storeProduct.value.filter((sp) => sp.shop === id).map((sp) => sp.product)
  console.log(choosedProductID)

  choosedProduct.value = products.value.filter((product) => choosedProductID.includes(product.id))
  console.log(choosedProduct.value)
}

function handlechangeFilter(event) {
  if (selected.value == 1) {
    choosedProduct.value.sort((a, b) => a.price - b.price)
  }

  if (selected.value == 2) {
    choosedProduct.value.sort((a, b) => b.price - a.price)
  }

  if (selected.value == 3) {
    choosedProduct.value.sort((a, b) => {
      return a.name.localeCompare(b.name)
    })
  }

  if (selected.value == 4) {
    choosedProduct.value.sort((a, b) => {
      return b.name.localeCompare(a.name)
    })
  }
}

const loadData = async () => {
  try {
    const response1 = await fetch('/stores.json');
    console.log(response1);

    if (response1.ok) {
      const data = await response1.json()
      stores.value = data.stores;
      console.log(stores.value);

        if (stores.value.length > 0) {
            choosedStore.value = stores.value[0].id;
        } else {
            console.warn('No stores found.');
        }

        console.log('Selected store ID:', choosedStore.value);
    } else {
        console.error('Failed to load data:', response1.statusText);
    }

    const response2 = await fetch('/products.json')
    if (response2.ok) {
      const data = await response2.json()
      products.value = data.products
      console.log(products.value)
    } else {
      console.error('Failed to load data:', response.statusText)
    }

    const responsenumberCardPerRow = await fetch('/storeProducts.json')
    if (responsenumberCardPerRow.ok) {
      const data = await responsenumberCardPerRow.json()
      storeProduct.value = data.shopProducts
      console.log(storeProduct.value)
    } else {
      console.error('Failed to load data:', response.statusText)
    }

    const choosedProductID = storeProduct.value
      .filter((sp) => sp.shop === choosedStore.value)
      .map((sp) => sp.product)
    console.log(choosedProductID)

    choosedProduct.value = products.value.filter((product) => choosedProductID.includes(product.id))
    console.log(choosedProduct.value)
  } catch (error) {
    console.error('Error fetching data:', error)
  }
}
loadData()
</script>

<script>
export default {
  data() {
    return {
      windowWidth: window.innerWidth,
      windowHeight: window.innerHeight,
      numberCardPerRow: ((window.innerWidth - 250) / 270) | 0
    }
  },
  methods: {
    updateWindowSize() {
      this.windowWidth = window.innerWidth
      this.windowHeight = window.innerHeight
      this.numberCardPerRow = ((window.innerWidth - 250) / 270) | 0
    }
  },
  mounted() {
    window.addEventListener('resize', this.updateWindowSize)
  },
  beforeUnmounted() {
    window.removeEventListener('resize', this.updateWindowSize)
  }
}
</script>

<template>
  <div class="sidebar">
    <div class="sidebar-header">MILK TEA STORE</div>
    <div
      v-for="item in stores"
      :key="item.id"
      class="sidebar-item"
      @click="handleChangeStore(item.id)"
    >
      {{ 'Stores ' + item.id }}
    </div>
  </div>

  <div class="content">
    <div class="store-heading">
      <a>{{ 'Stores ' + choosedStore + ' Menu' }} </a>
    </div>

    <div class="flex-container">
      <div class="filter">Filter</div>
      <div class="sort-filter flex-container">
        <span class="sorted-by"> Sorted by </span>
        <select v-model="selected" class="select" @change="handlechangeFilter">
          <option value="1">Price (ACS)</option>
          <option value="2">Price (DCS)</option>
          <option value="3">Name (ACS)</option>
          <option value="4">Name (DCS)</option>
        </select>
      </div>
    </div>
    <div
      v-for="j in ((choosedProduct.length / numberCardPerRow) | 0) + 1"
      :key="j"
      class="flex-nav"
    >
      <div v-for="i in numberCardPerRow" :key="i">
        <div v-if="(j - 1) * numberCardPerRow + i - 1 < choosedProduct.length" class="card">
          <div class="card-header">
            <div class="card-id">
              {{ `MT - ${choosedProduct[(j - 1) * numberCardPerRow + i - 1].id}` }}
            </div>
            <div class="card-title">
              {{ choosedProduct[(j - 1) * numberCardPerRow + i - 1].name }}
            </div>
          </div>
          <hr />
          <div class="card-toppings">
            <strong class="topping">Toppings:</strong>
            <div class="card-topping">
              {{ choosedProduct[(j - 1) * numberCardPerRow + i - 1].toppings }}
            </div>
          </div>

          <div class="card-footer">
            <span class="price">{{
              `$${choosedProduct[(j - 1) * numberCardPerRow + i - 1].price}`
            }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div></div>
</template>
