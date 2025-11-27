<script setup>
import { ref, computed, onMounted } from 'vue';

// --------------------- STATE ---------------------
const lessons = ref([]);             // all lessons from backend
const cart = ref([]);                // items added to cart
const showCart = ref(false);         // toggle between lessons view & cart view

const sortBy = ref('subject');       // 'subject' | 'location' | 'price' | 'space'
const sortOrder = ref('asc');        // 'asc' | 'desc'
const searchTerm = ref('');          // search as you type

const name = ref('');
const phone = ref('');
const checkoutMessage = ref('');

// --------------------- FETCH LESSONS ---------------------
async function fetchLessons() {
  try {
    const res = await fetch('https://express-backend-1g5j.onrender.com/lessons');
    const data = await res.json();
    
    lessons.value = data.map(l => ({
      ...l,
      space: Number(l.space)
    }));
  } catch (err) {
    console.error('Error fetching lessons:', err);
  }
}

onMounted(() => {
  fetchLessons();
});

// --------------------- SORT & FILTER ---------------------
const filteredLessons = computed(() => {
  const term = searchTerm.value.trim().toLowerCase();
  if (!term) return lessons.value;

  return lessons.value.filter(lesson => {
    const subject = String(lesson.subject ?? '').toLowerCase();
    const location = String(lesson.location ?? '').toLowerCase();
    const price = String(lesson.price ?? '').toLowerCase();
    const space = String(lesson.space ?? '').toLowerCase();

    return (
      subject.includes(term) ||
      location.includes(term) ||
      price.includes(term) ||
      space.includes(term)
    );
  });
});

const sortedAndFilteredLessons = computed(() => {
  const list = [...filteredLessons.value];
  const key = sortBy.value;
  const order = sortOrder.value === 'asc' ? 1 : -1;

  list.sort((a, b) => {
    let A = a[key];
    let B = b[key];

    if (typeof A === 'number' && typeof B === 'number') {
      return (A - B) * order;
    }

    A = String(A ?? '').toLowerCase();
    B = String(B ?? '').toLowerCase();

    if (A < B) return -1 * order;
    if (A > B) return 1 * order;
    return 0;
  });

  return list;
});

// --------------------- CART LOGIC ---------------------
function addToCart(lesson) {
  if (lesson.space > 0) {
    cart.value.push(lesson);
    lesson.space -= 1;
  }
}

function removeFromCart(index) {
  const item = cart.value[index];
  if (!item) return;

  const lesson = lessons.value.find(l => l._id === item._id);
  if (lesson) {
    lesson.space += 1;
  }

  cart.value.splice(index, 1);
}

const cartHasItems = computed(() => cart.value.length > 0);

function toggleCart() {
  showCart.value = !showCart.value;
  checkoutMessage.value = '';
}

// --------------------- VALIDATION ---------------------
const nameValid = computed(() => /^[A-Za-z\s]+$/.test(name.value.trim()));
const phoneValid = computed(() => /^[0-9]+$/.test(phone.value.trim()));

const canCheckout = computed(() => {
  return cartHasItems.value && nameValid.value && phoneValid.value;
});

// --------------------- CHECKOUT (POST /orders + PUT /lessons/:id) ---------------------
async function checkout() {
  if (!canCheckout.value) return;

  try {
    const summaryById = {};
    for (const item of cart.value) {
      const id = item._id;
      if (!summaryById[id]) {
        summaryById[id] = { lessonId: id, quantity: 0 };
      }
      summaryById[id].quantity += 1;
    }

    const items = Object.values(summaryById);

    const orderPayload = {
      name: name.value,
      phone: phone.value,
      items
    };

    // 1) POST /orders
    const res = await fetch('https://express-backend-1g5j.onrender.com/orders', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(orderPayload)
    });

    if (!res.ok) {
      throw new Error('Failed to save order');
    }

    // 2) PUT /lessons/:id to update space in DB
    for (const item of items) {
      const lesson = lessons.value.find(l => l._id === item.lessonId);
      if (!lesson) continue;

      await fetch(`https://express-backend-1g5j.onrender.com/lessons/${item.lessonId}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ space: lesson.space })
      });
    }

    checkoutMessage.value = `Order submitted for ${name.value} (${phone.value}).`;
    cart.value = [];
    name.value = '';
    phone.value = '';
    showCart.value = false;

    setTimeout(() => {
      checkoutMessage.value = '';
    }, 4000);

  } catch (err) {
    console.error('Checkout error:', err);
    checkoutMessage.value = 'There was a problem submitting your order. Please try again.';
  }
}
</script>

<template>
  <div class="app">
    <header class="header">
      <h1>After-School Lessons</h1>

      <div class="header-controls">
        <input
          v-model="searchTerm"
          type="text"
          placeholder="Search lessons..."
          class="search-input"
        />

        <select v-model="sortBy" class="select">
          <option value="subject">Subject</option>
          <option value="location">Location</option>
          <option value="price">Price</option>
          <option value="space">Spaces</option>
        </select>

        <select v-model="sortOrder" class="select">
          <option value="asc">Ascending</option>
          <option value="desc">Descending</option>
        </select>

        <button
          class="cart-toggle-btn"
          :disabled="!showCart && cart.length === 0"
          @click="toggleCart"
        >
          {{ showCart ? 'Back to Lessons' : 'View Cart' }}
          <span v-if="cart.length > 0 && !showCart">({{ cart.length }})</span>
        </button>
      </div>
    </header>

    <div v-if="checkoutMessage" class="success-banner">
      {{ checkoutMessage }}
    </div>

    <main v-if="!showCart" class="lessons-container">
      <div
        class="lesson-card"
        v-for="lesson in sortedAndFilteredLessons"
        :key="lesson._id"
      >
        <div class="lesson-image-wrapper" v-if="lesson.image">
          <img
            class="lesson-image"
            :src="`https://express-backend-1g5j.onrender.com/images/${lesson.image}`"
            :alt="lesson.subject"
          />
        </div>

        <h2 class="lesson-title">{{ lesson.subject }}</h2>
        <p class="lesson-text"><strong>Location:</strong> {{ lesson.location }}</p>
        <p class="lesson-text"><strong>Price:</strong> £{{ lesson.price }}</p>
        <p class="lesson-text">
          <strong>Spaces left:</strong> {{ lesson.space }}
        </p>

        <button
          class="add-btn"
          :disabled="lesson.space <= 0"
          @click="addToCart(lesson)"
        >
          {{ lesson.space > 0 ? 'Add to Cart' : 'Full' }}
        </button>
      </div>
    </main>

    <main v-else class="cart-container">
      <h2>Shopping Cart</h2>

      <div v-if="!cartHasItems" class="empty-cart">
        Your cart is empty.
      </div>

      <ul v-else class="cart-list">
        <li
          v-for="(item, index) in cart"
          :key="index"
          class="cart-item"
        >
          <span>{{ item.subject }} - £{{ item.price }}</span>
          <button class="remove-btn" @click="removeFromCart(index)">
            Remove
          </button>
        </li>
      </ul>

      <section class="checkout-section">
        <h3>Checkout</h3>

        <div class="form-group">
          <label for="name">Name (letters only)</label>
          <input id="name" v-model="name" type="text" class="input" />
          <small v-if="name && !nameValid" class="error">
            Name must contain letters and spaces only.
          </small>
        </div>

        <div class="form-group">
          <label for="phone">Phone (numbers only)</label>
          <input id="phone" v-model="phone" type="text" class="input" />
          <small v-if="phone && !phoneValid" class="error">
            Phone must contain digits only.
          </small>
        </div>

        <button
          class="checkout-btn"
          :disabled="!canCheckout"
          @click="checkout"
        >
          Checkout
        </button>

        <p v-if="checkoutMessage" class="checkout-message">
          {{ checkoutMessage }}
        </p>
      </section>
    </main>
  </div>
</template>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI',
    sans-serif;
}

.app {
  min-height: 100vh;
  background: #f5f5f5;
}

.header {
  padding: 16px 24px;
  background: #1e293b;
  color: white;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.header h1 {
  margin: 0;
}

.header-controls {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-items: center;
}

.search-input {
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #cbd5f5;
  min-width: 180px;
}

.select {
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #cbd5f5;
  background: white;
}

.cart-toggle-btn {
  padding: 6px 12px;
  border-radius: 6px;
  border: none;
  background: #f97316;
  color: white;
  cursor: pointer;
}

.cart-toggle-btn:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.lessons-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(230px, 1fr));
  gap: 16px;
  padding: 16px;
}

.lesson-card {
  background: white;
  border-radius: 10px;
  padding: 12px;
  box-shadow: 0 1px 4px rgba(15, 23, 42, 0.1);
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.lesson-image-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 8px;
}

.lesson-image {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 8px;
}

.lesson-title {
  margin: 0;
  font-size: 1.1rem;
}

.lesson-text {
  margin: 0;
  font-size: 0.9rem;
}

.add-btn {
  margin-top: auto;
  background: #22c55e;
  border: none;
  color: white;
  padding: 10px 0;
  font-size: 1rem;
  border-radius: 8px;
  cursor: pointer;
}

.add-btn:disabled {
  background: #9ca3af;
  cursor: not-allowed;
}

.cart-container {
  padding: 16px;
}

.cart-list {
  list-style: none;
  padding: 0;
  margin: 8px 0 16px;
}

.cart-item {
  display: flex;
  justify-content: space-between;
  background: white;
  margin-bottom: 6px;
  padding: 8px 10px;
  border-radius: 6px;
}

.remove-btn {
  border: none;
  background: #ef4444;
  color: white;
  border-radius: 4px;
  padding: 4px 8px;
  cursor: pointer;
}

.empty-cart {
  margin-bottom: 12px;
}

.checkout-section {
  background: white;
  padding: 12px;
  border-radius: 8px;
  max-width: 320px;
}

.form-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 8px;
}

.input {
  padding: 6px 8px;
  border-radius: 6px;
  border: 1px solid #cbd5e1;
}

.error {
  color: #dc2626;
  font-size: 0.8rem;
}

.checkout-btn {
  margin-top: 4px;
  padding: 6px 10px;
  border-radius: 6px;
  border: none;
  background: #3b82f6;
  color: white;
  cursor: pointer;
}

.checkout-btn:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.checkout-message {
  margin-top: 8px;
}

.success-banner {
  background: #22c55e;
  color: white;
  padding: 12px 16px;
  margin: 16px;
  border-radius: 6px;
  font-size: 1.1rem;
  font-weight: 600;
  text-align: center;
}
</style>
