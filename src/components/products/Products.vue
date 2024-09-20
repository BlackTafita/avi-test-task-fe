<script setup lang="ts">

import Product from "@/components/products/Product.vue";
import axios from "axios";
import { onMounted, ref } from "vue";

interface ProductsResponse {
  products: Product[];
  metadata: {
    items: number;
    offset: number;
  }
}

interface Product {
  name: string;
  description: string;
  media: {
    mainMedia: {
      image: {
        url: string;
      }
    }
  };
  price: {
    price: string;
    currency: string;
  }
}

let products = ref([] as Product[]);

async function fetchProductsList() {
  const fetchingProducts = await axios.get<ProductsResponse>('http://localhost:3000/products').then((res) => res.data);
  products.value = fetchingProducts.products;
}

onMounted(async () => {
  await fetchProductsList();
});

</script>

<template>
  <h1>Products Page</h1>
  <div class="content-block">
    <div class="product-list" v-if="products.length">
      <Product v-for="(product, index) in products" v-bind:product="product" v-bind:index="index"/>
    </div>
  </div>
</template>

<style scoped>
    .product-list {
      display: flex;
      flex-direction: row;
      align-items: center;
      justify-content: flex-start;
      break-after: auto;
      flex-wrap: wrap;
      max-width: 50vw;
    }
</style>