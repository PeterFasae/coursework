<script setup>
import { ref, computed } from 'vue';

// Reactive properties for managing lessons and cart
const lessons = ref([
  { id: 1, title: 'Introduction to Python', price: 20, category: 'Programming', rating: 4.5, slots: 5, image: 'src/assets/images/python.jpg' },
  { id: 2, title: 'Data Science Basics', price: 25, category: 'Data Science', rating: 4.7, slots: 8, image: 'src/assets/images/data_science.jpg' },
  { id: 3, title: 'Web Development with Vue', price: 18, category: 'Web Development', rating: 4.9, slots: 7, image: 'src/assets/images/vue.jpg' },
  { id: 4, title: 'Advanced JavaScript', price: 22, category: 'Programming', rating: 4.6, slots: 6, image: 'src/assets/images/javascript.jpg' },
  { id: 5, title: 'Introduction to AI', price: 30, category: 'Artificial Intelligence', rating: 4.8, slots: 10, image: 'src/assets/images/ai.jpg' },
  { id: 6, title: 'Digital Marketing Basics', price: 15, category: 'Marketing', rating: 4.3, slots: 12, image: 'src/assets/images/marketing.jpg' },
  { id: 7, title: 'Cloud Computing Fundamentals', price: 28, category: 'Cloud Computing', rating: 4.6, slots: 9, image: 'src/assets/images/cloud_computing.jpg' },
  { id: 8, title: 'Machine Learning Advanced', price: 35, category: 'Data Science', rating: 4.9, slots: 4, image: 'src/assets/images/machine_learning.jpg' },
  { id: 9, title: 'Blockchain Essentials', price: 32, category: 'Technology', rating: 4.4, slots: 5, image: 'src/assets/images/blockchain.jpg' },
  { id: 10, title: 'UI/UX Design Principles', price: 24, category: 'Design', rating: 4.7, slots: 10, image: 'src/assets/images/ui_ux.jpg' },
  { id: 11, title: 'React for Beginners', price: 20, category: 'Web Development', rating: 4.5, slots: 6, image: 'src/assets/images/react.jpg' },
  { id: 12, title: 'Cybersecurity Basics', price: 27, category: 'Security', rating: 4.6, slots: 8, image: 'src/assets/images/cybersecurity.jpg' }
]);
const cart = ref([]);
const showCart = ref(false);
const searchQuery = ref('');
const sortOption = ref({ attribute: 'title', order: 'asc' });
const customerDetails = ref({ name: '', phone: '' });
const validationErrors = ref({ name: false, phone: false });
const orderSubmitted = ref(false);

// Computed properties
const cartItemCount = computed(() => cart.value.length);
const cartTotal = computed(() => cart.value.reduce((total, lesson) => total + lesson.price, 0).toFixed(2));

// Functions
const addToCart = (lesson) => {
  if (lesson.slots > 0) {
    cart.value.push(lesson);
    lesson.slots -= 1;
  }
};

const removeFromCart = (lesson) => {
  const index = cart.value.indexOf(lesson);
  if (index !== -1) {
    cart.value.splice(index, 1);
    lesson.slots += 1;
  }
};

const toggleCartView = () => {
  showCart.value = !showCart.value;
};

const validateCustomerDetails = () => {
  validationErrors.value.name = !/^[a-zA-Z\s]+$/.test(customerDetails.value.name);
  validationErrors.value.phone = !/^\d{11,}$/.test(customerDetails.value.phone);
  return !validationErrors.value.name && !validationErrors.value.phone;
};

const checkout = () => {
  if (validateCustomerDetails()) {
    orderSubmitted.value = true;
    cart.value = [];  // Clear cart after checkout
  }
};

const filteredAndSortedLessons = computed(() => {
  return lessons.value
    .filter(lesson => lesson.title.toLowerCase().includes(searchQuery.value.toLowerCase()))
    .sort((a, b) => {
      let valueA = a[sortOption.value.attribute];
      let valueB = b[sortOption.value.attribute];

      if (typeof valueA === 'string') {
        valueA = valueA.toLowerCase();
        valueB = valueB.toLowerCase();
      }

      return sortOption.value.order === 'asc' ? (valueA > valueB ? 1 : -1) : (valueA < valueB ? 1 : -1);
    });
});
</script>

<template>
  <header class="header">
    <img src="./assets/images/lesson_logo.jpg" alt="Lessons Marketplace Logo">
    <h1>Welcome to the Lessons Marketplace</h1>
    <button class="btn btn-light" @click="toggleCartView">{{ showCart ? 'Continue Browsing' : `Cart (${cartItemCount})` }}</button>
  </header>

  <div class="container my-4" v-if="!showCart">
    <input type="text" class="form-control mb-3" v-model="searchQuery" placeholder="Search lessons...">
    <div class="row mb-4">
      <div class="col">
        <select class="form-control" v-model="sortOption.attribute">
          <option value="title">Title</option>
          <option value="price">Price</option>
          <option value="rating">Rating</option>
        </select>
      </div>
      <div class="col">
        <select class="form-control" v-model="sortOption.order">
          <option value="asc">Low to High</option>
          <option value="desc">High to Low</option>
        </select>
      </div>
    </div>

    <div class="row">
      <div class="col-md-4 lesson" v-for="lesson in filteredAndSortedLessons" :key="lesson.id">
        <div class="card mb-4">
          <img :src="lesson.image" class="card-img-top" :alt="`${lesson.title} image`" />
          <div class="card-body">
            <div class="d-flex justify-content-between align-items-center">
              <h5 class="card-title">{{ lesson.title }} ({{ lesson.category }})</h5>
              <p class="price">£{{ lesson.price.toFixed(2) }}</p>
            </div>
            <p>Rating: {{ lesson.rating }}</p>
            <button class="btn btn-danger" @click="addToCart(lesson)" :disabled="!lesson.slots">
              Enroll ({{ lesson.slots }} left)
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div v-if="showCart" class="checkout-page">
    <h2>Your Enrollments</h2>
    <div class="cart-items">
      <div v-for="lesson in cart" :key="lesson.id" class="lesson">
        <img :src="lesson.image" :alt="`${lesson.title} image`" />
        <div class="lesson-info">
          <div class="d-flex justify-content-between align-items-center">
            <p>{{ lesson.title }}</p>
            <p class="price">£{{ lesson.price.toFixed(2) }}</p>
          </div>
          <button class="btn btn-warning" @click="removeFromCart(lesson)">Remove</button>
        </div>
      </div>
    </div>

    <div class="checkout-inputs">
      <input type="text" class="form-control mb-2" v-model="customerDetails.name" placeholder="Enter your name">
      <input type="tel" class="form-control mb-2" v-model="customerDetails.phone" placeholder="Enter your phone number">
    </div>

    <span class="error-message" v-if="validationErrors.name">Only letters allowed for the name.</span>
    <span class="error-message" v-if="validationErrors.phone">Phone number must be 11 digits minimum.</span>

    <div class="cart-actions">
      <p><strong>Total:</strong> £{{ cartTotal }}</p>
      <button class="btn btn-success" @click="checkout" :disabled="cart.length === 0 || validationErrors.name || validationErrors.phone">Checkout</button>
    </div>

    <p v-if="orderSubmitted" class="success-message">Your order has been placed!</p>
  </div>
</template>

<style scoped>
body {
  font-family: Arial, sans-serif;
  background: #f9f9f9;
  color: #333;
}
.header {
  padding: 20px;
  background: #4B5563;
  color: white;
  text-align: center;
}
.header img {
  max-width: 150px;
  margin-bottom: 10px;
}
.lesson {
  border: 1px solid #ddd;
  background: #fff;
  padding: 10px;
  margin: 10px 0;
  border-radius: 8px;
  display: flex;
  align-items: center;
}
.lesson img {
  width: 100px;
  margin-right: 20px;
  border-radius: 8px;
}
.lesson-info {
  flex-grow: 1;
}
.price {
  color: #1F2937;
  font-size: 20px;
  font-weight: bold;
}
button {
  background: #1F2937;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 5px;
  font-size: 14px;
  cursor: pointer;
}
button:hover {
  background: #111827;
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
.cart-items .lesson {
  flex-direction: column;
  align-items: flex-start;
}
.cart-items .lesson button {
  margin-top: 10px;
}
.cart-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20px;
}
</style>
