<script setup lang="ts">

import { Delete } from "@element-plus/icons-vue";
import axios from "axios";
import { onMounted, reactive, ref } from "vue";

export interface ProductsResponse {
  products: ProductSchema[];
  metadata: {
    items: number;
    offset: number;
  }
}

export interface ProductSchema {
  _id: string;
  name: string;
  description: string;
  media: {
    mainMedia: {
      image: {
        url: string;
      }
    }
  };
  priceData: {
    price: string;
    currency: string;
  },
  productOptions: {
    name: string,
    choices: {
      value: string;
      description: string;
    }[]
  }[]
}

let products = ref([] as ProductSchema[]);
let editModalVisible = ref(false);
let modalTitle: 'Create' | 'Edit' = 'Create';
let optionsForm = reactive({
  input: [] as string[],
});
let form = reactive({
  _id: '',
  name: '',
  description: '',
  price: 0,
  variants: [
  ] as {
    name: string,
    options: string[],
  }[],
});

async function fetchProductsList() {
  products.value = await axios.get<ProductSchema[]>('http://localhost:3000/products').then((res) => res.data) || [];
}

function showEditModal(row: ProductSchema) {
  modalTitle = 'Edit';
  editModalVisible.value = true;
  form.name = row.name;
  form.description = row.description;
  form.price = Number(row.priceData.price);
  form._id = row._id;
  form.variants = row.productOptions.map((el) => ({
    name: el.name,
    options: el.choices.map((o) => o.value),
  }));
  optionsForm.input = row.productOptions.map(() => '');
}

function createNewProduct() {
  modalTitle = 'Create';
  editModalVisible.value = true;
}

function resetForm() {
  form = reactive({
    _id: '',
    name: '',
    description: '',
    price: 0,
    variants: [] as {
      name: string,
      options: string[],
    }[],
  });
}

const onSubmit = async () => {
  form.price = Number(form.price);
  if (form._id.length) {
    await editProductReq();
  } else {
    await createProductReq();
  }

  editModalVisible.value = false;
}

function addVariant() {
  form.variants.push({
    name: '',
    options: [],
  });
  optionsForm.input.push('');
}

function removeVariant(index: number) {
  form.variants.splice(index, 1);
}

function addVariantOption(variantKey: number) {
  const val = optionsForm.input[variantKey];
  form.variants[variantKey].options.push(val);
  optionsForm.input[variantKey] = '';
}

function removeOption(index: number, optionIndex: number) {
  optionsForm.input.splice(optionIndex, 1);
  form.variants[index].options.splice(optionIndex, 1);
}

async function createProductReq() {
  const newProduct = await axios.post<{
    product: ProductSchema,
  }>(`http://localhost:3000/products`, form).then((res) => res.data)

  resetForm();

  products.value.push(newProduct.product);
}
async function editProductReq() {
  const updateResponse = await axios.patch<{
    product: ProductSchema,
  }>(`http://localhost:3000/products/${form._id}`, form).then((res) => res.data)

  resetForm();

  const productKey = products.value.findIndex((el) => el._id === updateResponse.product._id);
  if (productKey >= 0) {
    products.value[productKey].name = updateResponse.product.name;
    products.value[productKey].description = updateResponse.product.description;
    products.value[productKey].priceData.price = updateResponse.product.priceData.price;
    products.value[productKey].productOptions = updateResponse.product.productOptions;
  }
}

async function deleteProductReq(id: string) {
  await axios.delete<{
    product: ProductSchema,
  }>(`http://localhost:3000/products/${id}`).then((res) => res.data);

  const deletedIndex = products.value.findIndex((el) => el._id === id);
  if (deletedIndex > -1) {
    products.value.splice(deletedIndex, 1);
  }
}

onMounted(async () => {
  await fetchProductsList();
});

</script>

<template>
  <h1>Products Page</h1>
  <div class="content-block" v-if="products.length">
    <div class="actions-row">
      <el-button @click="createNewProduct" type="primary">Create Product</el-button>
    </div>
    <el-table :data="products">
      <el-table-column label="Image">
        <template #default="scope">
          <img :src="scope?.row.media?.mainMedia?.image?.url" alt="Product Image" class="product-image">
        </template>
      </el-table-column>
      <el-table-column label="Title" property="name"/>
      <el-table-column label="Price">
        <template #default="scope">
          <span class="product-price">{{scope.row.priceData.price}} {{scope.row.priceData.currency}}</span>
        </template>
      </el-table-column>
      <el-table-column label="Description">
        <template #default="scope">
          <div class="product-description" v-html="scope.row.description"></div>
        </template>
      </el-table-column>
      <el-table-column label="Actions">
        <template #default="scope">
          <div class="product-actions">
            <el-button type="warning" @click="showEditModal(scope.row)">Edit</el-button>
            <el-button type="danger" @click="deleteProductReq(scope.row._id)">Delete</el-button>
          </div>
        </template>
      </el-table-column>
    </el-table>
  </div>

  <el-dialog
      v-model="editModalVisible"
      width="500"
  >
    <template #title>
      <span class="modal-title"> {{modalTitle}} Product </span>
    </template>
    <el-form :model="form">
      <el-row>
        <el-col :span="11">
          <el-form-item label="Product Name" label-position="top">
            <el-input v-model="form.name"/>
          </el-form-item>
        </el-col>
        <el-col :span="2"></el-col>
        <el-col :span="11">
          <el-form-item label="Price" label-position="top">
            <el-input v-model="form.price" type="number"/>
          </el-form-item>
        </el-col>
      </el-row>
      <el-form-item label="Description" label-position="top">
        <el-input v-model="form.description" type="textarea" resize="none"/>
      </el-form-item>
      <el-divider></el-divider>
      <div class="variants-list">
        <div class="variants-title">
          <span class="title">Variants</span>
          <el-link type="primary" @click="addVariant">Add Variants</el-link>
        </div>
        <div class="variants-block" v-for="(variant, index) of form.variants">
          <div class="top">
            <el-icon size="medium" @click="removeVariant(index)"><Delete /></el-icon>
          </div>
          <el-form-item label="Option Name" label-position="top">
            <el-input v-model="form.variants[index].name" />
          </el-form-item>

              <el-form-item label="Value" label-position="top">
                <el-row>
                  <el-col :span="16">
                    <el-input v-model="optionsForm.input[index]"/>
                  </el-col>
                  <el-col :span="2"></el-col>
                  <el-col :span="6" class="option-btn">
                    <el-button type="primary" :disabled="!optionsForm.input[index].length" @click="addVariantOption(index)">Add</el-button>
                  </el-col>
                </el-row>
              </el-form-item>
              <div class="options-list">
                <el-tag v-for="(option, optIndex) of variant.options" :key="index" closable type="primary" @close="removeOption(index, optIndex)">
                  {{option}}
                </el-tag>
              </div>
        </div>
      </div>
    </el-form>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="editModalVisible = false">Cancel</el-button>
        <el-button type="primary" @click="onSubmit">
          {{ modalTitle }}
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>

<style scoped>

    .content-block {
      max-width: 80vw;
      min-width: 60vw;
    }

    .product-image {
      width: 100px;
    }

    .product-price {
      font-weight: bold;
    }
    .product-description {
      font-size: 10px;
    }
    .product-actions {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: space-between;

      * {
        margin: 0.5em 0;
      }
    }

    .actions-row {
      display: flex;
      width: 100%;
      align-items: center;
      justify-content: flex-end;
      margin-bottom: 1em;
    }

    .modal-title {
      color: black;
      font-weight: bold;
      font-size: 16px;
    }

    .variants-title {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 1em;

      .title {
        font-size: 14px;
        font-weight: bold;
      }
    }

    .variants-block {
      .top {
        display: flex;
        justify-content: flex-end;
        align-items: center;
      }
    }

    .option-btn {
      display: flex;
      align-items: flex-end;
      justify-content: center;
    }
</style>