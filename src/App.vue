<script setup>
import { ref, computed } from "vue";
import Shirt from "./components/Shirt/index.vue";
import Pant from "./components/Pant/index.vue";
import Logo from "./assets/logo.svg";
import User from "./assets/user.svg";
import Cart from "./assets/cart.svg";
import Search from "./assets/search.svg";

const currentTab = ref("shirt");
const cart = ref([]);
const cartCount = ref(0);
const searchQuery = ref(""); // Biến lưu giá trị tìm kiếm
const tempQuery = ref("");
const email = ref("");
const password = ref("");


// Các biến trạng thái cho modal
const isLoginVisible = ref(false);
const isRegisterVisible = ref(false);
const isConfirm = ref(false);
const accounts = ref(JSON.parse(localStorage.getItem("accounts")) || []);
const currentUser = ref(null);
const dangnhap = ref("dangnhap");

// Hiển thị modal đăng nhập/đăng ký
const toggleLoginModal = (v) => {
  if(v == "dangnhap") {
    isLoginVisible.value = !isLoginVisible.value;
  } else {
    isRegisterVisible.value = !isRegisterVisible.value;
    isLoginVisible.value = !isLoginVisible.value;
  }
};

const toggleRegisterModal = () => {
  isRegisterVisible.value = !isRegisterVisible.value;
  isLoginVisible.value = !isLoginVisible.value;
};

// Đăng ký tài khoản
const registerAccount = (email, password) => {
  const accountExists = accounts.value.some((acc) => acc.email === email);
  if (accountExists) {
    alert("Email đã tồn tại!");
    return;
  }
  accounts.value.push({ email, password });
  localStorage.setItem("accounts", JSON.stringify(accounts.value));
  alert("Đăng ký thành công!");
  isRegisterVisible.value = !isRegisterVisible.value;
};

// Đăng nhập
const loginAccount = (email, password) => {
  const account = accounts.value.find(
    (acc) => acc.email === email && acc.password === password
  );
  if (account) {
    currentUser.value = account;
    alert("Đăng nhập thành công!");
    isLoginVisible.value = !isLoginVisible.value;
  } else {
    alert("Sai email hoặc mật khẩu!");
  }
};

// Đăng xuất
const logoutAccount = () => {
  currentUser.value = null;
  alert("Đã đăng xuất!");
};

// kết thúc đăng nhập, đăng xuất

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

const handleSearch = () => {
  searchQuery.value = tempQuery.value;
};

const handleConfirm = () => {
  isConfirm.value = !isConfirm.value
  isPopoverVisible.value = !isPopoverVisible.value;
}
// Tính tổng tiền
const totalPrice = computed(() => {
  return cart.value.reduce((total, item) => total + item.price * item.quantity, 0);
});

// Hàm xử lý thanh toán
const processPayment = () => {
  cartCount.value = 0
  alert('Thanh toán thành công!');
  cart.value = [];
  saveCartToLocalStorage();
  isConfirm.value = !isConfirm.value
};
</script>

<template>
  <div>
    <div class="container">
      <!-- Menu điều hướng -->
       <nav class="nav-bar">
        <div class="d-flex justify-content-between">
          <ul class="nav">
            <li class="nav-item me-3">
              <div>
                <img :src="Logo" width="40px" />
              </div>
            </li>
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
            <li class="nav-item">
              <button
                class="nav-link"
                :class="{ active: currentTab === 'gioithieu' }"
                @click="changeTab('gioithieu')"
              >
                Giới thiệu
              </button>
            </li>
            <li class="nav-item">
              <button
                class="nav-link"
                :class="{ active: currentTab === 'lienhe' }"
                @click="changeTab('lienhe')"
              >
                Liên hệ
              </button>
            </li>
          </ul>

          
          <div class="d-flex  position-relative">
            <input class="inputSearch form-control me-2" v-model="tempQuery" placeholder="Bạn đang tìm gì" />
            <button class="btn-search" @click="handleSearch(searchQuery)">
              <img :src="Search" width="20px" />
            </button>
          </div>
          
          <!-- Icon giỏ hàng -->
          <div class="d-flex">
              <button
                class=" position-relative"
                @click="togglePopover"
              >
              <img :src="Cart" width="20px" />
                <span
                  class="position-absolute top-20 translate-middle badge bg-dark rounded-pill"
                >
                  {{ cartCount }}
                </span>
              </button>
              <div>
                <!-- Nút Đăng Xuất -->
                <div v-if="currentUser" class="d-flex align-items-center justify-content-center pt-2">
                  <div>{{ currentUser.email }}, </div>
                  <div class="btn-logout" @click="logoutAccount">Đăng Xuất</div>
                </div>
                <button v-else @click="toggleLoginModal(dangnhap)">
                  <img :src="User" />
                </button>
              </div>
          </div>
        </div>
      </nav>

      <!-- Modal Đăng Nhập -->
      <!-- Nền tối khi form hiển thị -->
      <div v-if="isLoginVisible" class="auth-overlay" @click="toggleLoginForm"></div>

      <div v-if="isLoginVisible" class="auth-form">
        <h2>Đăng Nhập</h2>
        <form @submit.prevent="loginAccount(email, password)">
          <input v-model="email" type="text" placeholder="Email" />
          <input v-model="password" type="password" placeholder="Mật khẩu" />
          <button type="submit">Đăng Nhập</button>
        </form>
            <p class="text-center mt-4" type="button" @click="toggleRegisterModal">
              Chưa có tài khoản? Đăng Ký
            </p>
      </div>

    <!-- Modal Đăng Ký -->
    <!-- Nền tối khi form hiển thị -->
    <div v-if="isRegisterVisible" class="auth-overlay" @click="toggleLoginForm"></div>

    <!-- Form đăng nhập/đăng ký -->
    <div v-if="isRegisterVisible" class="auth-form">
      <h2>Đăng Ký</h2>
      <form @submit.prevent="registerAccount(email, password)">
        <input v-model="email" type="text" placeholder="Email" />
        <input v-model="password" type="password" placeholder="Mật khẩu" />
        <button type="submit">Đăng Ký</button>
      </form>
          <p class="text-center mt-4" type="button" @click="toggleLoginModal">
            Đã có tài khoản? Đăng Nhập
          </p>
    </div>


      <!-- Popover hiển thị giỏ hàng sản phẩm -->
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
        <button v-if="cart.length > 0" class="btn btn-add-to-cart w-100" @click="handleConfirm">Xác nhận thanh toán</button>
      </div>
    </div>

    <!-- Hiển thị xác nhận thanh toán giỏ hàng -->
    <div v-if="isConfirm" class="auth-overlay"></div>

    <div v-if="isConfirm" class="auth-form" style="width: 50%;">
      <h2>Xác nhận thanh toán</h2>
      <div class="row">
        <div class="col-md-7">
          <div v-if="cart.length > 0">
            <div v-for="item in cart" :key="item.id" class="popover-item">
              <div class="popover-left align-items-center">
                <div class="popover-image">
                  <img :src="item.imageUrl" alt="product image" />
                </div>
                  <h5 class="pt-1">{{ item.name }}</h5>
                  <p class="pt-3">{{ item.price }} VND</p>
              </div>
            </div>
            <div class="total-price">
              <h4>Tổng tiền: {{ totalPrice }} VND</h4>
            </div>
            <button class="mt-4" @click="processPayment">Xác nhận thanh toán</button>
          </div>
          <div v-else>
            <p>Giỏ hàng của bạn hiện tại trống.</p>
          </div>
        </div>
        <div class="col-md-5">
          <h5>Địa chỉ</h5>
          <textarea class="form-control" placeholder="Vui lòng nhập" rows="4"></textarea>
          <input class="form-control" placeholder="Nhập mã khuyến mãi (nếu có)" rows="4"></input>
        </div>
      </div>
      <div class="text-end mt-4">
        <button style="width: 70px;" class="btn btn-secondary" @click="isConfirm = !isConfirm">Đóng</button>
      </div>
    </div>

    <!-- Slider ảnh -->
    <div id="carouselExampleIndicators" class="carousel slide" data-bs-ride="carousel">
      <div class="carousel-indicators">
        <button type="button" data-bs-target="#carouselExampleIndicators" data-bs-slide-to="0" class="active" aria-current="true" aria-label="Slide 1"></button>
        <button type="button" data-bs-target="#carouselExampleIndicators" data-bs-slide-to="1" aria-label="Slide 2"></button>
      </div>
      <div class="carousel-inner">
        <div class="carousel-item active">
          <img src="https://file.hstatic.net/200000891157/file/banner-pc.jpg" class="d-block w-100" alt="...">
        </div>
        <div class="carousel-item">
          <img src="https://file.hstatic.net/200000887901/file/banner-aristino-pc.jpg" class="d-block w-100" alt="...">
        </div>
      </div>
      <button class="carousel-control-prev" type="button" data-bs-target="#carouselExampleIndicators" data-bs-slide="prev">
        <span class="carousel-control-prev-icon" aria-hidden="true"></span>
        <span class="visually-hidden">Previous</span>
      </button>
      <button class="carousel-control-next" type="button" data-bs-target="#carouselExampleIndicators" data-bs-slide="next">
        <span class="carousel-control-next-icon" aria-hidden="true"></span>
        <span class="visually-hidden">Next</span>
      </button>
    </div>

    <!-- Nội dung -->
    <div class="container">
      <div class="tab-content mt-4">
        <div v-if="currentTab === 'shirt'" class="tab-pane active">
          <Shirt :searchQuery="searchQuery" @add-to-cart="addToCart" />
        </div>
        <div v-if="currentTab === 'pant'" class="tab-pane active">
          <Pant :searchQuery="searchQuery" @add-to-cart="addToCart" />
        </div>
        <div v-if="currentTab === 'gioithieu'" class="tab-pane active">
          <div>
            <h4>Giới thiệu</h4>
            <p>
              Với sự phát triển của ngành công nghiệp thời trang Việt Nam, việc sử dụng nội dung chất lượng cao như content bán quần áo đã trở thành một cách hiệu quả để tiếp cận và thu hút khách hàng. Bằng cách cung cấp hướng dẫn mix đồ, chia sẻ xu hướng thời trang, và bí quyết chăm sóc quần áo, các shop thời trang có thể tạo dựng lòng tin và tăng doanh số bán hàng. Hãy tận dụng những xu hướng thời trang mùa Xuân Hè và mùa Thu Đông để tạo ra nội dung hấp dẫn, thu hút và phù hợp với nhu cầu người tiêu dùng
              Bạn muốn tham khảo các mẫu content bán quần áo hay để gia tăng tỉ lệ tương tác cũng như tỉ lệ chuyển đổi cho doanh nghiệp thì bài viết này dành cho bạn. Dưới đây ABC Digi gợi ý cho bạn 15+ ý tưởng content bán quần áo cũng như các mẫu content có sẵn để bạn tham khảo nhanh chóng.
            </p>
          </div>
        </div>
        <div v-if="currentTab === 'lienhe'" class="tab-pane active">
          <div>
            <h4>Bản đồ cửa hàng</h4>
            <p>
              Khách hàng có thể tìm kiếm và đo khoảng cách ngay tại trang liên hệ thay vì phải mở Google Map và thực hiện các thao tác mất thời gian. Ngoài trụ sở chính, bạn cũng cần ghim địa chỉ các chi nhánh để khách hàng chủ động theo dõi.
            </p>
            <iframe src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d3724.1051250207915!2d105.85000457476916!3d21.028479287789214!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x3135ac69f861af3f%3A0x331250d72cd2fa28!2zSOG7kyBHxrDGoW0gKEhvYcyAbiBLacOqzIFtKQ!5e0!3m2!1svi!2s!4v1731950608173!5m2!1svi!2s" width="100%" height="750" style="border:0;" allowfullscreen="" loading="lazy" referrerpolicy="no-referrer-when-downgrade"></iframe>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.inputSearch {
  width: 500px;
  border: 1px solid #262626;
  height: 36px;
  padding-right: 50px;
}
.btn-search {
  position: absolute;
  top: 0;
  right: 0;
  border: 1px solid #262626;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 0;
  border-top-right-radius: 0.375rem;
  border-bottom-right-radius: 0.375rem;
}
.nav-bar {
  padding: 20px 0;
}
.nav-item {
  height: 32px;
}
.nav-link {
  color: #262626;
  font-weight: 600;
}
.nav-link.active {
  border-bottom: 2px solid #262626;
}
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

/* đăng nhập, đăng xuất */
/* Form đăng nhập/đăng ký */
.auth-form {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: 350px;
  z-index: 9999;
}

/* Phần tiêu đề của form */
.auth-form h2 {
  text-align: center;
  margin-bottom: 20px;
  color: #333;
}

/* Các input trong form */
.auth-form input {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 14px;
}
.btn-logout {
  font-weight: 600;
  margin-left: 4px;
  cursor: pointer;
}
/* Các nút trong form */
.auth-form button {
  width: 100%;
  padding: 10px;
  background-color: #262626;
  color: #fff;
  border: none;
  border-radius: 5px;
  font-size: 14px;
  cursor: pointer;
}

.auth-form button:hover {
  background-color: #444;
}

/* Để form đăng nhập/đăng ký bị ẩn khi không cần thiết */
.auth-form.hidden {
  display: none;
}

/* Phần nền tối khi popup hiển thị */
.auth-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5); /* Độ mờ cho nền tối */
  z-index: 9998;
}

</style>
./components/Shirt.vue./components/Shirt/Shirt.vue
