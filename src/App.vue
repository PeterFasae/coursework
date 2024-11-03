<script setup>
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'
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
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>

