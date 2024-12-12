<template>
  <div class="order-detail" v-if="orderExists">
    <div class="action-button">
      <button @click="completeOrder" class="button">Complete Order</button>
    </div>
    <br />
    <div class="order-header">
      <p><b>Order ID:</b> {{ order.orderId }}</p>
      <p><b>Customer ID:</b> {{ order.customerId }}</p>
      <p><b>Status:</b> {{ order.status }}</p>
    </div>
    <div class="order-items">
      <table>
        <thead>
          <tr>
            <th>Product ID</th>
            <th>Product Name</th>
            <th>Quantity</th>
            <th>Price</th>
            <th>Total</th>
          </tr>
        </thead>
        <tr v-for="item in order.items" :key="item.productId">
          <td><router-link :to="`/product/${item.productId}`">{{ item.productId }}</router-link></td>
          <td>{{ productLookup(item.productId) }}</td>
          <td>{{ item.quantity }}</td>
          <td>{{ item.price.toFixed(2) }}</td>
          <td>{{ (item.quantity * item.price).toFixed(2) }}</td>
        </tr>
        <tfoot>
          <tr>
            <td colspan="4" style="text-align: right;"><b>Total:</b></td>
            <td><b>{{ orderTotal() }}</b></td>
          </tr>
        </tfoot>
      </table>
    </div>
  </div>
  <div class="order-detail" v-else>
    <h3>Oops! That order was not found...</h3>
  </div>
</template>

<script>
export default {
  name: 'OrderDetail',
  props: ['orders', 'products'],
  emits: ['completeOrder'],
  data() {
    return {
      order: null,
    };
  },
  computed: {
    orderExists() {
      return !!this.order;
    },
  },
  mounted() {
    this.getOrder();
  },
  methods: {
    getOrder() {
      this.order = this.orders.find(order => order.orderId === this.$route.params.id);

      if (!this.order) {
        fetch(`/makeline/order/${this.$route.params.id}`)
          .then(response => {
            if (!response.ok) {
              throw new Error('Network response was not ok');
            }
            if (response.status === 204) {
              return null;
            }
            return response.json();
          })
          .then(data => {
            if (data) {
              this.order = data;
            } else {
              console.log('No orders from server');
            }
          })
          .catch(error => console.error(error));
      }

      return this.order;
    },
    completeOrder() {
      this.$emit('completeOrder', this.order.orderId);
    },
    productLookup(id) {
      return this.products.find(product => product.id === id).name;
    },
    orderTotal() {
      let total = 0;
      this.order.items.forEach(item => {
        total += item.price * item.quantity;
      });
      return total.toFixed(2);
    },
  },
};
</script>

<style scoped>
a {
  color: #0046be; /* BestBuy blue for links */
  text-decoration: underline;
}

.order-detail {
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

tfoot td {
  font-weight: bold;
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
</style>
