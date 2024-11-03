<script setup>
import {ref, computed} from 'vue';
const products = ref ([
      { id: 1, name: 'Dark Chocolate', price: 5, category: 'Dark', rating: 4.5, stock: 10, image: 'images/dark_chocolate.jpg' },
      { id: 2, name: 'Milk Chocolate', price: 4, category: 'Milk', rating: 4.2, stock: 15, image: 'images/milk_chocolate.jpg' },
      { id: 3, name: 'White Chocolate', price: 6, category: 'White', rating: 4.8, stock: 8, image: 'images/white_chocolate.jpg' },
      { id: 4, name: 'Chocolate Truffle', price: 8, category: 'Gourmet', rating: 4.7, stock: 5, image: 'images/truffle.jpg' },
    ]);
const cart = ref([]);
const cartPage = ref(false);
const searchQuery = ref('');
const sortAttribute = ref('name');
const sortOrder = ref('asc');
const customerName = ref('');
const customerPhone = ref('');
const orderSubmitted =  ref(false);
const nameError = ref(false);
const phoneError = ref(false);

const cartItemCount = computed(() => cart.value.length);
const cartTotal = computed(() => cart.value.reduce((total, item) => total + item.price, 0).toFixed(2));

function addToCart(product) {
  if (product.stock > 0) {
    cart.value.push(product);
    product.stock -= 1; // Reduce stock by 1
  }
}
function removeFromCart(product) {
  const index = cart.value.indexOf(product);
  if (index !== -1) {
    cart.value.splice(index, 1);
    product.stock += 1; // Restock the product
  }
}

function toggleCartPage() {
  cartPage.value = !cartPage.value;
}

function validateName() {
  const namePattern = /^[a-zA-Z\s]*$/;
  nameError.value = !namePattern.test(customerName.value);
}

function validatePhone() {
  const phonePattern = /^\d{11,}$/;
  phoneError.value = !phonePattern.test(customerPhone.value);
}
</script>

<template>
  <header class="header">
    <img src="../images/logo.png" alt="Chocolate Boutique Logo">
    <h1>Welcome to the Chocolate Boutique</h1>
    <button class="btn btn-light" @click="toggleCartPage">{{ cartPage ? 'Continue Shopping' : `Cart (${cartItemCount})` }}</button>
  </header>

  <div class="container my-4" v-if="!cartPage">
    <input type="text" class="form-control mb-3" v-model="searchQuery" placeholder="Search chocolates...">
    <div class="row mb-4">
      <div class="col">
        <select class="form-control" v-model="sortAttribute">
          <option value="name">Name</option>
          <option value="price">Price</option>
        </select>
      </div>
      <div class="col">
        <select class="form-control" v-model="sortOrder">
          <option value="asc">Low to High</option>
          <option value="desc">High to Low</option>
        </select>
      </div>
    </div>

    <div class="row">
      <!-- <div class="col-md-4" v-for="product in filteredAndSortedProducts" :key="product.id" class="product"> -->
      <div class="col-md-4 product" v-for="product in filteredAndSortedProducts" :key="product.id">
        <div class="card mb-4">
          <img :src="product.image" class="card-img-top" :alt="`${product.name} image`" />
          <div class="card-body">
            <div class="d-flex justify-content-between align-items-center">
              <h5 class="card-title">{{ product.name }} ({{ product.category }})</h5>
              <p class="price">£{{ product.price.toFixed(2) }}</p>
            </div>
            <p>Rating: {{ product.rating }}</p>
            <button class="btn btn-danger" @click="addToCart(product)" :disabled="!product.stock">
              Add to Cart ({{ product.stock }} left)
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div v-if="cartPage" class="checkout-page">
    <h2>Your Shopping Cart</h2>
    <div class="cart-items">
      <div v-for="item in cart" :key="item.id" class="product">
        <img :src="item.image" :alt="`${item.name} image`" />
        <div class="product-info">
          <div class="d-flex justify-content-between align-items-center">
            <p>{{ item.name }}</p>
            <p class="price">£{{ item.price.toFixed(2) }}</p>
          </div>
          <button class="btn btn-warning" @click="removeFromCart(item)">Remove from Cart</button>
        </div>
      </div>
    </div>

    <div class="checkout-inputs">
      <input type="text" class="form-control mb-2" v-model="customerName" placeholder="Enter your name" @input="validateName">
      <input type="tel" class="form-control mb-2" v-model="customerPhone" placeholder="Enter your phone number" @input="validatePhone">
    </div>

    <span class="error-message" v-if="nameError">Only letters allowed for the name.</span>
    <span class="error-message" v-if="phoneError">Phone number must be 11 digits minimum.</span>

    <div class="cart-actions">
      <p><strong>Total:</strong> £{{ cartTotal }}</p>
      <button class="btn btn-success" @click="checkout" :disabled="cart.length === 0 || nameError || phoneError">Checkout</button>
    </div>

    <p v-if="orderSubmitted" class="success-message">Your order has been placed!</p>
  </div>
</template>


<style scoped>
body {
  font-family: Arial, sans-serif;
  background: #f4f4f4;
  color: #333;
}
.header {
  padding: 20px;
  background: #5B3E31;
  color: white;
  text-align: center;
}
.header img {
  max-width: 150px;
  margin-bottom: 10px;
}
.product {
  border: 1px solid #ddd;
  background: #fff;
  padding: 10px;
  margin: 10px 0;
  border-radius: 8px;
  display: flex;
  align-items: center;
}
.product img {
  width: 100px;
  margin-right: 20px;
  border-radius: 8px;
}
.product-info {
  flex-grow: 1;
}
.price {
  color: #E63946;
  font-size: 20px;
  font-weight: bold;
}
button {
  background: #E63946;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 5px;
  font-size: 14px;
  cursor: pointer;
}
button:hover {
  background: #D62828;
}
.checkout-page {
  padding: 20px;
  max-width: 800px;
  margin: auto;
  background: white;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}
.error-message {
  color: #E63946;
  font-size: 12px;
}
.cart-items .product {
  flex-direction: column;
  align-items: flex-start;
}
.cart-items .product button {
  margin-top: 10px;
}
.cart-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20px;
}
</style>