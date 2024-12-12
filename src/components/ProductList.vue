<template>
  <div class="action-button">
    <router-link to="/product/add">
      <button class="button">Add Product</button>
    </router-link>
  </div>
  <div class="product-list">
    <table>
      <thead>
        <tr>
          <th>Product ID</th>
          <th>Product Name</th>
          <th>Product Description</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="product in products" :key="product.productId">
          <td><router-link :to="`/product/${product.id}`">{{ product.id }}</router-link></td>
          <td>{{ product.name }}</td>
          <td>{{ product.description }}</td>
          <td>${{ product.price.toFixed(2) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'ProductList',
  props: ['products'],
  mounted() {
    this.$emit('getProducts');
  }
};
</script>

<style scoped>
a {
  color: #0046be; /* BestBuy blue for links */
  text-decoration: underline;
}

.action-button {
  text-align: right;
  margin-bottom: 1rem;
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

.product-list {
  text-align: left;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
  border: 1px solid #ddd;
}

th,
td {
  padding: 8px;
  text-align: left;
}

th {
  background-color: #0046be; /* BestBuy blue */
  color: white;
}

tbody tr:nth-child(odd) {
  background-color: #f9f9f9;
}
</style>
