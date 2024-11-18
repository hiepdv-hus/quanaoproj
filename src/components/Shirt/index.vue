<template>
  <div class="product-list">
    <h1 class="product-title">Áo nam</h1>

    <div class="product-header d-flex align-items-center justify-content-between align-items-center mb-3">
      <p>Tổng số sản phẩm: {{ products.length }}</p>

      <!-- Dropdown sắp xếp -->
      <select v-model="sortOption" class="form-select w-auto">
        <option value="default">Mặc định</option>
        <option value="bestSeller">Bán chạy nhất</option>
        <option value="newest">Sản phẩm mới</option>
        <option value="highestPrice">Giá cao nhất</option>
        <option value="lowestPrice">Giá thấp nhất</option>
      </select>
    </div>
    <!-- Hiển thị danh sách sản phẩm -->
    <div class="product-lst row">
      <div
        class="col-lg-3 col-md-6 col-12"
        v-for="product in currentProducts"
        :key="product.id"
      >
        <div class="product-item">
          <img
            :src="product.imageUrl"
            alt="Product Image"
            class="product-image"
            @click="showProductDetails(product)"
          />
          <div class="product-details" @click="showProductDetails(product)">
            <h2>{{ product.name }}</h2>
            <p>
              Giá:
              <span class="product-price">{{ product.price }} VND</span>
              <span class="product-discount-price">
                {{ product.discountPrice }} VND</span
              >
            </p>
            <div class="d-flex justify-content-between px-3">
              <p v-if="product.quantity == 0 || product.quantity == ''" style="font-style: italic;">Hết hàng</p>
              <p v-else class="product-rating">Số lượng: {{ product.quantity }}</p>
              <p class="product-rating">Đánh giá: ⭐ {{ product.rating }}</p>
            </div>
          </div>
          <button
            class="btn btn-add-to-cart w-100"
            @click="$emit('add-to-cart', product)"
          >
            Thêm vào Giỏ hàng
          </button>
        </div>
      </div>
    </div>

    <!-- Popup chi tiết sản phẩm -->
    <div class="modal fade" id="productModal" tabindex="-1" aria-hidden="true">
      <div class="modal-dialog modal-md">
        <div class="modal-content">
          <div class="modal-header">
            <h3 class="modal-title">{{ selectedProduct.name }}</h3>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <img
              :src="selectedProduct.imageUrl"
              alt="Product Image"
              class="img-fluid mb-3"
            />
            <p><strong>Giá gốc:</strong> {{ selectedProduct.price }} VND</p>
            <p class="product-discount-price">
              <strong>Giá sau giảm:</strong>
              {{ selectedProduct.discountPrice }} VND
            </p>
            <p><strong>Đánh giá:</strong> ⭐ {{ selectedProduct.rating }}</p>
            <p><strong>Số lượng:</strong> {{ selectedProduct.quantity }}</p>
            <p><strong>Mô tả:</strong> {{ selectedProduct.description }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Phân trang -->
    <nav aria-label="Page navigation">
      <ul class="pagination">
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <a class="page-link" href="#" @click.prevent="changePage(1)">Đầu</a>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <a
            class="page-link"
            href="#"
            @click.prevent="changePage(currentPage - 1)"
            >Trước</a
          >
        </li>

        <li
          v-for="page in totalPages"
          :key="page"
          class="page-item"
          :class="{ active: page === currentPage }"
        >
          <a class="page-link" href="#" @click.prevent="changePage(page)">{{
            page
          }}</a>
        </li>

        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
          <a
            class="page-link"
            href="#"
            @click.prevent="changePage(currentPage + 1)"
            >Sau</a
          >
        </li>
        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
          <a class="page-link" href="#" @click.prevent="changePage(totalPages)"
            >Cuối</a
          >
        </li>
      </ul>
    </nav>
  </div>
</template>

<script setup>
import { ref, computed, defineProps } from "vue";
import Shirt from "./Shirt.json";
import "../style.css";

const count = ref(0);
const products = ref([]);
const currentPage = ref(1);
const productsPerPage = 8;
const selectedProduct = ref({});
const sortOption = ref("default");

const props = defineProps({
  searchQuery: {
    type: String,
    default: "", // Giá trị mặc định là chuỗi rỗng
  },
});

console.log('props', props);


const getList = async () => {
  const response = await Shirt;
  products.value = response;
};
getList();

// Hiển thị chi tiết sản phẩm
const showProductDetails = (product) => {
  selectedProduct.value = product;
  const modal = new bootstrap.Modal(document.getElementById("productModal"));
  modal.show();
};

// Tổng số trang
const totalPages = computed(() => {
  return Math.ceil(products.value.length / productsPerPage);
});

// Hàm thay đổi trang
const changePage = (page) => {
  if (page < 1) page = 1;
  if (page > totalPages.value) page = totalPages.value;
  currentPage.value = page;
};

const sortedProducts = computed(() => {
  let filtered = products.value.filter((product) =>
    product.name.toLowerCase().includes(props.searchQuery.toLowerCase())
  );

  switch (sortOption.value) {
    case "bestSeller":
      filtered.sort((a, b) => b.sales - a.sales); // Sắp xếp theo số lượng bán giảm dần
      break;
    case "newest":
      filtered.sort((a, b) => new Date(b.releaseDate) - new Date(a.releaseDate)); // Sắp xếp theo ngày phát hành mới nhất
      break;
    case "highestPrice":
      filtered.sort((a, b) => b.price - a.price); // Sắp xếp theo giá giảm dần
      break;
    case "lowestPrice":
      filtered.sort((a, b) => a.price - b.price); // Sắp xếp theo giá tăng dần
      break;
    default:
      break;
  }
  return filtered;
});


const currentProducts = computed(() => {
  const startIndex = (currentPage.value - 1) * productsPerPage;
  const endIndex = startIndex + productsPerPage;
  return sortedProducts.value.slice(startIndex, endIndex);
});
</script>
