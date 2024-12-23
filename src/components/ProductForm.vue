<template>
  <div class="action-button">
    <button @click="saveProduct" class="button">Save Product</button>
  </div>
  <br />
  <div v-if="showValidationErrors" class="error">
    <br />
    <ul v-for="error in validationErrors" :key="error">
      <li>{{ error }}</li>
    </ul>
  </div>
  <div class="product-form">
    <table>
      <tr>
        <td><label for="product-name">Name</label></td>
        <td><input id="product-name" placeholder="Product Name" v-model="product.name" /></td>
      </tr>

      <tr>
        <td><label for="product-price">Price</label></td>
        <td><input id="product-price" placeholder="Product Price" v-model="product.price" type="number" step="0.01" /></td>
      </tr>

      <tr>
        <td><label for="product-tags">Keywords</label></td>
        <td><input id="product-tags" placeholder="Product Keywords" v-model="product.tags" /></td>
      </tr>

      <tr>
        <td><label for="product-description">Description</label></td>
        <td>
          <textarea rows="8" id="product-description" placeholder="Product Description" v-model="product.description"></textarea>
          <input type="hidden" id="product-id" placeholder="Product ID" v-model="product.id" />
        </td>
        <td>
          <button @click="generateDescription" class="ai-button" v-show="aiCapabilities.includes('description')">Ask AI Assistant</button>
        </td>
      </tr>

      <tr>
        <td><label for="product-image">Image</label></td>
        <td>
          <input id="product-image-text" placeholder="Product Image" v-model="product.image" v-show="!aiCapabilities.includes('image')" />
          <div id="product-image-container" class="image-container" :class="{ loading: isLoadingImage }" v-show="aiCapabilities.includes('image')">
            <img v-if="product.image" :src="product.image" alt="Product Image" />
            <div class="overlay">{{ overlayText }}</div>
          </div>
        </td>
        <td>
          <button id="product-image-btn" @click="generateImage" class="ai-button" v-show="aiCapabilities.includes('image')">Generate Image</button>
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
const aiServiceUrl = '/ai/';
const productServiceUrl = '/product/';

export default {
  name: 'ProductForm',
  props: ['products'],
  emits: ['addProductsToList', 'updateProductInList'],
  data() {
    return {
      product: {
        id: 0,
        name: '',
        image: '/placeholder.png',
        description: '',
        price: 0.0,
        tags: []
      },
      aiCapabilities: [],
      showValidationErrors: false,
      isLoadingImage: false,
      overlayText: ''
    };
  },
  mounted() {
    if (this.$route.params.id) {
      const product = this.products.find(product => product.id == this.$route.params.id);
      this.product = Object.assign({}, product);
      if (!this.product.tags) {
        this.product.tags = [];
      }
    }

    fetch(`${aiServiceUrl}health`)
      .then(response => response.json())
      .then(data => {
        if (data.status === 'ok') {
          this.aiCapabilities = data.capabilities;
        }
      })
      .catch(error => console.error(error));
  },
  computed: {
    validationErrors() {
      let errors = [];
      if (this.product.name.length === 0) errors.push('Please enter a value for the name field');
      if (this.product.description.length === 0) errors.push('Please enter a value for the description field');
      if (this.product.price <= 0) errors.push('Please enter a value greater than 0 for the price field');
      return errors;
    }
  },
  methods: {
    generateDescription() {
      if (this.product.tags.length === 0) {
        alert('Please enter a value for the keywords field');
        return;
      }
      const intervalId = this.waitForAI();
      const requestBody = {
        name: this.product.name,
        tags: this.product.tags.split(',').map(tag => tag.trim())
      };
      fetch(`${aiServiceUrl}generate/description`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(requestBody)
      })
        .then(response => response.json())
        .then(product => {
          this.product.description = product.description;
        })
        .finally(() => clearInterval(intervalId));
    },
    generateImage() {
      if (this.product.description === '') {
        alert('Please enter a product description');
        return;
      }
      this.isLoadingImage = true;
      this.overlayText = 'Drawing...';
      const requestBody = {
        name: this.product.name,
        description: this.product.description
      };
      fetch(`${aiServiceUrl}generate/image`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(requestBody)
      })
        .then(response => response.json())
        .then(product => {
          this.overlayText = 'Downloading...';
          this.product.image = product.image;
        })
        .finally(() => {
          this.isLoadingImage = false;
        });
    },
    waitForAI() {
      let dots = '';
      const intervalId = setInterval(() => {
        dots += '.';
        this.product.description = `Thinking${dots}`;
      }, 500);
      return intervalId;
    },
    saveProduct() {
      if (this.validationErrors.length > 0) {
        this.showValidationErrors = true;
        return;
      }
      const method = this.$route.path.includes('add') ? 'POST' : 'PUT';
      this.product.price = parseFloat(this.product.price);
      fetch(`${productServiceUrl}`, {
        method: method,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(this.product)
      })
        .then(response => response.json())
        .then(product => {
          alert('Product saved successfully');
          if (method === 'PUT') this.$emit('updateProductInList', this.product);
          else this.$emit('addProductsToList', product);
          this.$router.push(`/product/${product.id}`);
        });
    }
  }
};
</script>

<style scoped>
.error ul {
  color: #ff0000;
}

button {
  padding: 10px 20px;
  background-color: #0046be;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #003bb3;
}

table {
  width: 100%;
}

textarea {
  width: 100%;
}
</style>
