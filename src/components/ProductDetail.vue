<template>
  <div class="action-button">
    <router-link :to="`/product/${this.$route.params.id}/edit`">
      <button class="button">Edit Product</button>
    </router-link>
  </div>
  <br />
  <div class="product-detail" v-if="productExists">
    <div class="product-image">
      <img :src="product.image" alt="Product Image">
    </div>
    <div class="product-info">
      <h2>{{ product.name }} - ${{ product.price }}</h2>
      <small>Product ID: {{ product.id }}</small>
      <p>{{ product.description }}</p>
    </div>
  </div>
  <div class="product-detail" v-else>
    <h3>Product not found</h3>
  </div>
</template>

<script>
export default {
  name: 'ProductDetail',
  props: ['products'],
  computed: {
    product() {
      return this.products.find(product => product.id == this.$route.params.id);
    },
    productExists() {
      return !!this.product;
    }
  },
};
</script>

<style scoped>
a {
  color: #0046be; /* BestBuy blue for links */
  text-decoration: underline;
}

.action-button {
  text-align: right;
}

button {
  padding: 10px 20px;
  background-color: #0046be; /* BestBuy blue */
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #003bb3; /* Darker shade for hover */
}

.product-detail {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  gap: 1rem;
  margin: 2rem auto;
  text-align: left;
  max-width: 800px;
}

.product-image {
  flex: 1;
  margin-right: 20px;
}

.product-image img {
  width: 100%;
  height: auto;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.product-info {
  flex: 1;
  text-align: left;
}

.product-info h2 {
  font-size: 24px;
  margin-bottom: 10px;
  color: #333;
}

.product-info small {
  display: block;
  margin-bottom: 10px;
  color: #555;
}

.product-info p {
  font-size: 16px;
  margin-bottom: 20px;
  color: #666;
}

@media (max-width: 768px) {
  .product-detail {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .product-image {
    margin-right: 0;
  }

  .product-info {
    text-align: center;
  }
}
</style>
