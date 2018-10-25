<template>
  <div id="app">
    <div v-if="showModal">
      <!-- If showModal is true dispaly div -->
      <app-checkout :cartArr="cartArray" :cartTotal="cartTotal" @showCheckout="showCheckout" @clearCart="clearCheckout"></app-checkout>
    </div>
    <div  v-bind:class="{modal: showModal}">
      <!-- Sets class = modal if showModal = true -->
    <div class="fixed">
      <br />
    <div class="ui huge header">Vue Store</div>
    <!-- Navigation Bar -->
    <div class="ui inverted menu">
        <a class="item" @click="loadProducts">Home</a>
        <a class="item" @click="filterDepartment('Mens')">Mens</a>
        <a class="item" @click="filterDepartment('Womens')">Womens</a>
        <a class="item" @click="filterDepartment('Kids')">Kids</a>
        <div class="right menu">
          <a class="item" @click="showCart">
            <!--Toggles cart on and off-->
            {{ cartSize }}
            <i class="large in cart icon"></i>
          </a>
          <div v-if="show" class="cartPopup">
            <app-cart :cartSize="cartSize" :cartTotal="cartTotal" :cartArr="cartArray" @showCheckout="showCheckout" @clearCart="clearCart" @removeFromCart="removeFromCart($event)"
             @decreaseQuantity="decreaseQuantity($event)" @increaseQuantity="increaseQuantity($event)" @showCart="showCart"></app-cart>
            <!--:cart="cartArray" binds the cart 'prop' in Cart component to cartArray in Home. Whatever goes in the Home cartArray is now in Cart cartArr
                @clearCart="clearCart" When the clearCart method is called in the Cart component it "emits" an event. "@clearCart" listens for that event. When the event fires it call the clearCart method in the Home component.
                @removeFromCart="removeFromCart($event)" Same as above but passes the product that is clicked in the cart to the method-->
          </div>
      </div>
    </div>
  </div>
  <br/><br/><br/><br/>
  <div class="users margin">
    <div class="ui two column grid">
      <div class="ui three wide column">
        <div>
        <!-- Search -->
        <div class="ui icon input">
          <input v-model="searchBox" @input="search" type="text" placeholder="I am searching for...">
          <i class="search link icon"></i>
        </div>
        <!-- Start Category of Checkbox Items -->
        <div class="ui medium header">CATEGORY</div>
        <div class="ui list alignLeft">
          <div class="ui checkbox" >
            <input type="radio" tabindex="0" id="All" name="filter" v-on:click="loadProducts"  checked="checked">
            <label for="All">All</label>
          </div>
          <br />
          <div class="ui checkbox" >
            <input type="radio" tabindex="1" id="Acces" name="filter" v-on:click="filterCategory('A')">
            <label for="Acces">Accessories</label>
          </div>
          <br />
          <div class="ui checkbox" >
            <input type="radio" tabindex="2" id="C&J" name="filter" v-on:click="filterCategory('B')">
            <label for="C&J">Coats & Jackets</label>
          </div>
          <br />
          <div class="ui checkbox" >
            <input type="radio" tabindex="3" id="H&S" name="filter" v-on:click="filterCategory('C')">
            <label for="H&S">Hoodies & Sweats</label>
          </div>
          <br />
          <div class="ui checkbox" >
            <input type="radio" tabindex="4" id="J&T" name="filter" v-on:click="filterCategory('D')">
            <label for="J&T">Jeans & Trousers</label>
          </div>
          <br />
          <div class="ui checkbox" >
            <input type="radio" tabindex="5" id="J&C" name="filter" v-on:click="filterCategory('E')">
            <label for="J&C">Jumpers & Cardigans</label>
          </div>
          <br />
          <div class="ui checkbox" >
            <input type="radio" tabindex="6" id="S&R" name="filter" v-on:click="filterCategory('F')">
            <label for="S&R">Shoes & Runners</label>
          </div>
          <br />
          <div class="ui checkbox" >
            <input type="radio" tabindex="7" id="S&S" name="filter" v-on:click="filterCategory('G')">
            <label for="S&S">Suits & Shirts</label>
          </div>
        </div>
      </div>
    </div>
      <!-- End Category list of Checkbox Items -->
      <!-- Start Product List-->
      <div class="ui thirteen wide column three column grid">
        <div class="item column" v-for="product in filteredProducts">
          <div class="ui segment">
            <div class="ui small image">
              <img :src="product.imgURL">
            </div>
            <div class="content">
              <div class="header">{{product.title}}</div>
              <div class="meta">
                <div>
                  <span class="size">size: {{product.size}}</span>
                </div>
                <span class="price">{{product.price | currency('€')}}</span>
                <br/>
                <!-- Display Product Quantity in Stock -->
                <span v-if="product.stock == 0" class="stock">Out of Stock</span>
                <span v-else-if="product.stock < 10" class="stock">Hurry only {{product.stock}} left!</span>
                <span v-else-if="product.stock >= 10"><br /></span>
              </div>
              <!-- Add to Cart -->
              <div class="extra">
                <button  class="ui right floated black button" v-if="product.stock > 0" v-on:click="addToCart(product)">
                  Add To Cart
                  <i class="right chevron icon"></i>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- End Product List-->
    </div>
  </div>
</div>
</div>
</template>

<script>
  import Cart from './Cart.vue';
  import Checkout from './Checkout.vue';

  export default {
    name: 'Home',
    data() {
      return {
        show: false,
        showModal: false,
        searchBox: null,
        cartTotal: 0,
        filteredProducts: [],
        products: [], // Products are inserted on created
        cartArray: [] // End Cart
      }
    }, // End data

    components: {
        'app-cart': Cart,
        'app-checkout': Checkout
    },

    mounted: function () {// When the app is ready load the products
      this.loadProducts();
    },
      methods: {
        addToCart: function (product) {// Passes the product to the cart
          this.noItems ++;
          var found = false;
          product.stock -=1;
          for (var i = 0; i < this.cartArray.length; i++) {
            if(this.cartArray[i].sku === product.sku)
            {
              product.quantity ++;
              found = true;
              break;
            }
          }

          if(!found)
          {
            this.cartArray.push(product);
            product.quantity = 1;
          }
          this.getCartTotal();
        },
        clearCart: function () {// Removes all products from the cart and restocks them
          for (var i = 0; i < this.cartArray.length; i++) {
            for (var j = 0; j < this.filteredProducts.length; j++) {
              if(this.cartArray[i].sku === this.filteredProducts[j].sku)
              {
                this.filteredProducts[j].stock += this.filteredProducts[j].quantity;
              }
            }
          }
          this.cartArray = [];
          this.getCartTotal();
        },
        clearCheckout: function () {// Removes all products from the cart but does no restock them
          this.cartArray = [];
          this.getCartTotal();
        },
        removeFromCart: function (product) {// Removes the selected product from the cart
          for (var i = 0; i < this.cartArray.length; i++) {
            if (this.cartArray[i].sku == (product.sku))
            {
              product.stock += product.quantity;
              this.cartArray.splice(i, 1);
            }
          }
          this.getCartTotal();
        },
        decreaseQuantity: function (product) {// Decrements the qunatity of selected product in the Cart
          product.quantity --;
          product.stock ++;
          if (product.quantity == 0)
          {
            for (var i = 0; i < this.cartArray.length; i++) {
              if (this.cartArray[i].sku == (product.sku))
              {
                this.cartArray.splice(i, 1);
              }
            }
          }
          this.getCartTotal();
        },
        increaseQuantity: function (product) {// Increments the quantity of selected product in the Cart
          if (product.stock > 0) {
            product.quantity ++;
            product.stock --;
            this.getCartTotal();
          }
        },
        loadProducts: function() { // Loads All Products
          this.filteredProducts =[];
          for (var i = 0; i < this.products.length; i++) {
            {
              this.filteredProducts.push(this.products[i]);
            }
          }
          this.getCartTotal();
        },
        filterDepartment: function (department) { // Filters products displayed by Department
          this.searchBox = null;
          this.filteredProducts =[];
          for (var i = 0; i < this.products.length; i++) {
            if (this.products[i].department.includes(department))
            {
              this.filteredProducts.push(this.products[i]);
            }
          }
        },
        filterCategory: function (category) { // Filters products displayed by Category
          this.searchBox = null;
          this.filteredProducts =[];
          for (var i = 0; i < this.products.length; i++) {
            if (this.products[i].sku.includes(category))
            {
              this.filteredProducts.push(this.products[i]);
            }
          }
        },
        search: function () { // Searches all Products when text changes
          this.filteredProducts =[];
          for (var i = 0; i < this.products.length; i++) {
            if (this.products[i].title.toUpperCase().includes(this.searchBox.toUpperCase()))
            {
              this.filteredProducts.push(this.products[i]);
            }
          }
        },
        getCartTotal: function () {
          this.cartTotal = 0;
          for (var i = 0; i < this.cartArray.length; i++) {
            this.cartTotal += this.cartArray[i].price * this.cartArray[i].quantity;
          }
        },
        showCheckout() {// Reverses the bool value
          this.showModal = !this.showModal;
        },
        showCart() {
          this.show = !this.show;
        }
      }, // End Methods
      computed: {
          cartSize: function () {// Calculates the number of products in the Cart
            var total = 0;
            for (var i = 0; i < this.cartArray.length; i++) {
              total += this.cartArray[i].quantity;
            }
            return total;
          }
      },
      created: function () {// Imports products from db.JSON using restful API
        this.$http.get('http://localhost:3000/products')
        .then(function(response) {
        this.products = response.data;
        this.loadProducts();
      });
    } // End Created
  }
  </script>

<style scoped>
.modal {
  background-color: white;
   filter:alpha(opacity=10); /* IE */
   opacity: 0.1; /* Safari, Opera */
   -moz-opacity:0.10; /* FireFox */
}
.alignLeft {
  text-align: left;
}

.cartPopup {
      right: 0em;
      position: fixed;
      text-align: right;
      background: rgba(0,0,0,0.85);
      color: white;
      z-index: 2;
}

.fixed {
  right: 0em;
  top: 0em;
  background-color: white;
  width: 97.5vw;
  position: fixed;
  z-index: 2;
}

.margin {
  margin-right: auto;
  margin-left: auto;
  margin-bottom: auto;
  margin-top: 20px;
  width: 95%;
}

.stock {
  color: red;
  font-size: 12px;
}
</style>
