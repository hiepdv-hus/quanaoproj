<template>
  <div class="product-list">
    <h1 class="product-title">Quần nam</h1>

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
            <p class="product-rating">Đánh giá: ⭐ {{ product.rating }}</p>
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
            <p><strong>Giá:</strong> {{ selectedProduct.price }} VND</p>
            <p>
              <strong>Giá giảm:</strong> {{ selectedProduct.discountPrice }} VND
            </p>
            <p><strong>Đánh giá:</strong> ⭐ {{ selectedProduct.rating }}</p>
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
import { ref, computed } from "vue";
import Shirt from "./Pant.json";
import "../style.css";

const count = ref(0);
const products = ref([]);
const currentPage = ref(1);
const productsPerPage = 8;
const selectedProduct = ref({});

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

// Cập nhật dữ liệu hiện tại của trang
const currentProducts = computed(() => {
  const startIndex = (currentPage.value - 1) * productsPerPage;
  const endIndex = startIndex + productsPerPage;
  return products.value.slice(startIndex, endIndex);
});

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
</script>
