<script setup lang="ts">
const props = defineProps(['product', 'index']);
</script>

<template>
  <el-card class="product" :key="index">
    <template #header>
      <span class="product-name">{{product.name}}</span>
    </template>
    <img :src="product?.media?.mainMedia?.image?.url" alt="Product Image" class="product-image">
    <template #footer>
      <div class="product-description" v-html="product.description"></div>
      <div class="product-price">{{product.price.price}} {{product.price.currency}}</div>
      <div class="product-actions">
        <el-button type="warning">Edit</el-button>
        <el-button type="danger">Delete</el-button>
      </div>
    </template>
  </el-card>
</template>

<style scoped>
.product {
  margin: 1em;

  .product-name {
    font-weight: bold;
  }

  .product-image {
    width: 150px;
  }

  .product-price {
    margin-bottom: 1em;
    font-weight: bold;
  }

  .product-description {
    margin: 1em 0;
  }
}
</style>