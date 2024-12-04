<script setup>
import { ref, computed, reactive, watch } from 'vue';
import { lessons as lessonData } from './lessons.js';
import './styles.css';

const lessons = ref(lessonData);
const cart = ref([]);
const showCart = ref(false);
const searchQuery = ref('');
const sortOption = ref({ attribute: 'subject', order: 'asc' });
const customerDetails = reactive({ name: '', phone: '' });
const validationErrors = reactive({ name: false, phone: false });
const orderSubmitted = ref(false);
const isSubmitting = ref(false); // Tracks the form submission status

const cartItemCount = computed(() => cart.value.length);
const cartTotal = computed(() => cart.value.reduce((total, lesson) => total + lesson.price, 0).toFixed(2));

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
  if (showCart.value) {
    showCart.value = false;
  } else if (cart.value.length > 0) {
    showCart.value = true;
  }
};

const validateCustomerDetails = () => {
  validationErrors.name = !/^[a-zA-Z\s]+$/.test(customerDetails.name);
  validationErrors.phone = !/^\d+$/.test(customerDetails.phone);
  console.log("Validation - Name:", validationErrors.name, "Phone:", validationErrors.phone);
  return !validationErrors.name && !validationErrors.phone;
};

watch(() => customerDetails.name, () => {
  validationErrors.name = !/^[a-zA-Z\s]+$/.test(customerDetails.name);
});

watch(() => customerDetails.phone, () => {
  validationErrors.phone = !/^\d+$/.test(customerDetails.phone);
});

const submitForm = async () => {
  if (!validateCustomerDetails()) {
    return;
  }

  isSubmitting.value = true;

  const order = {
    customer: {
      name: customerDetails.name,
      phone: customerDetails.phone,
    },
    items: cart.value.map((lesson) => ({
      id: lesson.id,
      subject: lesson.subject,
      price: lesson.price,
      quantity: 1, // Assuming 1 for simplicity
    })),
    total: cartTotal.value,
  };

  try {
    const response = await fetch('https://coursework-backend.onrender.com/api/orders', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(order),
    });

    if (!response.ok) {
      throw new Error('Failed to submit order');
    }

    const data = await response.json();
    alert(data.message || 'Order placed successfully!');

    // Reset form and cart
    cart.value = [];
    customerDetails.name = '';
    customerDetails.phone = '';
    orderSubmitted.value = true;
  } catch (error) {
    alert('Failed to submit order. Please try again.');
    console.error(error);
  } finally {
    isSubmitting.value = false;
    showCart.value = false;
  }
};

// const checkout = () => {
//   if (validateCustomerDetails()) {
//     orderSubmitted.value = true;
//     cart.value = [];  
//   }
// };

// const filteredAndSortedLessons = computed(() => {
//   return lessons.value
//     .filter(lesson => lesson.subject.toLowerCase().includes(searchQuery.value.toLowerCase()))
//     .sort((a, b) => {
//       let valueA = a[sortOption.value.attribute];
//       let valueB = b[sortOption.value.attribute];

//       if (typeof valueA === 'string') {
//         valueA = valueA.toLowerCase();
//         valueB = valueB.toLowerCase();
//       }

//       return sortOption.value.order === 'asc' ? (valueA > valueB ? 1 : -1) : (valueA < valueB ? 1 : -1);
//     });
// });


// search function
const filteredAndSortedLessons = computed(() => {
  return lessons.value
    .filter(lesson => {
      // Check if search query matches any of the lesson attributes
      const matchesSubject = lesson.subject.toLowerCase().includes(searchQuery.value.toLowerCase());
      const matchesPrice = lesson.price.toString().includes(searchQuery.value);
      const matchesLocation = lesson.location.toLowerCase().includes(searchQuery.value.toLowerCase());
      const matchesSlots = lesson.slots.toString().includes(searchQuery.value);

      // Return true if any of the attributes match the search query
      return matchesSubject || matchesPrice || matchesLocation || matchesSlots;
    })
    .sort((a, b) => {
      let valueA = a[sortOption.value.attribute];
      let valueB = b[sortOption.value.attribute];

      // Normalize values to lowercase if they are strings for sorting
      if (typeof valueA === 'string') {
        valueA = valueA.toLowerCase();
        valueB = valueB.toLowerCase();
      }

      // Sort based on selected attribute and order (asc or desc)
      return sortOption.value.order === 'asc' ? (valueA > valueB ? 1 : -1) : (valueA < valueB ? 1 : -1);
    });
});

</script>

<template>
  <header class="header">
    <img src="/images/lesson_logo.jpg" alt="Lessons Marketplace Logo">
    <h1>Welcome to the Lessons Marketplace</h1>
    <button :class="cartItemCount ? 'button-active' : 'button-disabled'" @click="toggleCartView">
  {{ showCart ? 'Continue Browsing' : `Cart (${cartItemCount})` }}
</button>

  </header>

  <div class="container my-4" v-if="!showCart">
    <!-- <input type="text" class="form-control mb-3" v-model="searchQuery" placeholder="Search lessons..."> -->
    <input type="text" class="form-control mb-3" v-model="searchQuery" placeholder="Search lessons...">

    <div class="row mb-4">
      <div class="col">
        <select class="form-control" v-model="sortOption.attribute">
          <option value="subject">Subject</option>
          <option value="price">Price</option>
          <option value="location">Location</option>
          <option value="slots">Slots</option>
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
          <img :src="lesson.image" class="card-img-top" :alt="`${lesson.subject} image`" />
          <div class="card-body">
            <div class="d-flex justify-content-between align-items-center">
              <h5 class="card-subject">{{ lesson.subject }} ({{ lesson.category }})</h5>
              <p class="price">£{{ lesson.price.toFixed(2) }}</p>
            </div>
            <p>Location: {{ lesson.location }}</p>
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
        <img :src="lesson.image" :alt="`${lesson.subject} image`" />
        <div class="lesson-info">
          <div class="d-flex justify-content-between align-items-center">
            <p>{{ lesson.subject }}</p>
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
    <span class="error-message" v-if="validationErrors.phone">Only digits are allowed.</span>

    <!-- <div class="cart-actions">
      <p><strong>Total:</strong> £{{ cartTotal }}</p>
      <button class="btn btn-success" @click="checkout" :disabled="cartItemCount < 1 || validationErrors.name || validationErrors.phone">Checkout</button>
    </div> -->

    <div class="cart-actions">
      <p><strong>Total:</strong> £{{ cartTotal }}</p>
      <button
        class="btn btn-success"
        @click="submitForm"
        :disabled="isSubmitting || cartItemCount < 1 || validationErrors.name || validationErrors.phone"
      >
        {{ isSubmitting ? 'Submitting...' : 'Checkout' }}
      </button>
    </div>

    <p v-if="orderSubmitted" class="success-message">Your order has been placed!</p>
  </div>
</template>