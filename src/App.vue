<script setup>
import { ref } from "vue";
import Shirt from "./components/Shirt/index.vue";
import Pant from "./components/Pant/index.vue";
import Logo from "./assets/logo.svg";

const currentTab = ref("shirt");
const cart = ref([]);
const cartCount = ref(0);

// Hàm chuyển tab
const changeTab = (tab) => {
  currentTab.value = tab;
};

// Hàm thêm sản phẩm vào giỏ hàng
const addToCart = (product) => {
  const existingProduct = cart.value.find((item) => item.id === product.id);
  if (existingProduct) {
    // Nếu sản phẩm đã có trong giỏ -> Tăng số lượng
    existingProduct.quantity += 1;
  } else {
    // Nếu sản phẩm chưa có trong giỏ -> Thêm mới
    cart.value.push({ ...product, quantity: 1 });
  }
  updateCartCount();
  saveCartToLocalStorage();
};

// Cập nhật số lượng sản phẩm trong giỏ
const updateCartCount = () => {
  cartCount.value = cart.value.reduce(
    (total, item) => total + item.quantity,
    0
  );
};

// Lưu giỏ hàng vào localStorage
const saveCartToLocalStorage = () => {
  localStorage.setItem("cart", JSON.stringify(cart.value));
};

// Load giỏ hàng từ localStorage
const loadCartFromLocalStorage = () => {
  const savedCart = localStorage.getItem("cart");
  if (savedCart) {
    cart.value = JSON.parse(savedCart);
    updateCartCount();
  }
};

// Gọi hàm load khi khởi tạo
loadCartFromLocalStorage();

// Hàm xóa sản phẩm khỏi giỏ
const removeProductFromCart = (productId) => {
  cart.value = cart.value.filter((item) => item.id !== productId);
  updateCartCount();
  saveCartToLocalStorage();
};

// Điều kiện để hiển thị popover
const isPopoverVisible = ref(false);
const togglePopover = () => {
  isPopoverVisible.value = !isPopoverVisible.value;
};
</script>

<template>
  <div class="container">
    <!-- Menu điều hướng -->
    <div class="d-flex justify-content-between">
      <div>
        <img :src="Logo" width="30px" />
      </div>
      <ul class="nav nav-tabs">
        <li class="nav-item">
          <button
            class="nav-link"
            :class="{ active: currentTab === 'shirt' }"
            @click="changeTab('shirt')"
          >
            Áo
          </button>
        </li>
        <li class="nav-item">
          <button
            class="nav-link"
            :class="{ active: currentTab === 'pant' }"
            @click="changeTab('pant')"
          >
            Quần
          </button>
        </li>
        <li class="nav-item ms-auto ps-4">
          <!-- Icon giỏ hàng -->
          <button
            class="btn btn-primary position-relative"
            @click="togglePopover"
          >
            🛒
            <span
              class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger"
            >
              {{ cartCount }}
            </span>
          </button>
        </li>
      </ul>
    </div>

    <!-- Popover hiển thị danh sách sản phẩm -->
    <div v-if="isPopoverVisible" class="popover-container position-absolute">
      <p>Danh sách giỏ hàng</p>
      <div v-for="item in cart" :key="item.id" class="popover-item">
        <div class="popover-left">
          <div class="popover-image">
            <img :src="item.imageUrl" alt="product image" />
          </div>
          <div class="ms-2">
            <h5>{{ item.name }}</h5>
            <p>{{ item.price }} VND</p>
          </div>
        </div>
        <button @click="removeProductFromCart(item.id)" class="btn">
          <i class="fa fa-trash-o" style="font-size: 24px; color: red"></i>
        </button>
      </div>
    </div>

    <!-- Nội dung -->
    <div class="tab-content mt-4">
      <div v-if="currentTab === 'shirt'" class="tab-pane active">
        <Shirt @add-to-cart="addToCart" />
      </div>
      <div v-if="currentTab === 'pant'" class="tab-pane active">
        <Pant @add-to-cart="addToCart" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

.popover-image {
  width: 60px;
  height: 60px;
}
.popover-image img {
  width: 100%;
}
.popover-container {
  background-color: #fff;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
  width: 300px;
  max-height: 400px;
  overflow-y: auto;
  top: 60px; /* Can be adjusted based on layout */
  right: 0;
  padding: 10px;
  z-index: 9999;
  border-radius: 5px;
}

.popover-item {
  margin-bottom: 10px;
  display: flex;
  justify-content: space-between;
  gap: 10px;
}
.popover-item .popover-left {
  display: flex;
  gap: 10px;
}

.popover-item .btn-danger {
  font-size: 12px;
}
</style>
./components/Shirt.vue./components/Shirt/Shirt.vue
