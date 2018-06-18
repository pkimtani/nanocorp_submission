<template>
    <v-app id="inspire">
        <v-toolbar fixed app>
            <v-toolbar-title>
                <img src="public/logo-dark.png" style="max-width: 45%; padding-top: 2%;">
            </v-toolbar-title>
            <v-text-field
                    flat
                    prepend-icon="search"
                    label="Search"
                    class="hidden-sm-and-down mt-2"
                    v-model="searchedText"
            ></v-text-field>
            <v-spacer></v-spacer>
            <v-badge left color="green lighten-2">
                <span slot="badge">{{ products.length }}</span>
                <v-icon large color="grey lighten-1">shopping_cart</v-icon>
            </v-badge>
            &nbsp;&nbsp;
            <v-icon color="grey lighten-1" large>attach_money</v-icon>
            <span class="medium title pr-2">{{ finalAmount }}</span>
        </v-toolbar>
        <v-content>
          <v-container grid-list-md>
              <v-layout row wrap>
                  <v-flex v-for="product in displayProducts"
                          :key="product.id" xs12 sm6 md3 lg3>
                      <v-card @mouseover="mouseOver = product.id"
                              @mouseout="mouseOver = 0">
                          <v-card-media :src="cardMedia(product)" height="400px">
                              <v-container fill-height fluid>
                                  <v-layout fill-height>
                                      <v-flex xs12 align-end flexbox>
                                          <v-btn style="background-color: #1ABC9C;"
                                                 depressed
                                                 v-if="product.discount_text != 0"
                                                 small>
                                              {{ product.discount_text }}
                                          </v-btn>
                                      </v-flex>
                                  </v-layout>
                              </v-container>
                              <v-btn style="margin-top: 365px;"
                                     depressed
                                     v-if="mouseOver === product.id"
                                     @click="removeItem(product)"
                                     absolute>
                                  <v-icon small>delete</v-icon>&nbsp;Delete
                              </v-btn>
                              <v-btn style="margin-top: 365px;"
                                     v-if="mouseOver === product.id"
                                     depressed>
                                  <v-icon small>search</v-icon>&nbsp;Quick View
                              </v-btn>
                          </v-card-media>
                          <v-card-title primary-title>
                              <div>
                                  <h3 class="medium mb-0">{{ product.name }}</h3>
                                  <div v-if="product.discount_price > 0">
                                      <strike>
                                          $&nbsp;{{ product.price }}
                                      </strike>
                                      &nbsp;
                                      <span style="font-size: 18px; color: #1ABC9C; font-weight: 600;">
                                          $&nbsp;{{ product.discount_price }}
                                      </span>
                                  </div>
                                  <div v-else>
                                      <span style="font-size: 18px; font-weight: 600;">
                                          $&nbsp;{{ product.price }}
                                      </span>
                                  </div>
                              </div>
                          </v-card-title>
                      </v-card>
                  </v-flex>
                  <v-flex v-if="this.products.length === 0" xs12 sm6 md3 lg3>
                      <v-card>
                          <v-card-media src="public/images/no.png" height="400px" width="200px;">
                          </v-card-media>
                          <v-card-title primary-title>
                              <div>
                                  <h3 class="medium mb-0">No More Products</h3>
                                  <div>
                                      $&nbsp;0
                                  </div>
                              </div>
                          </v-card-title>
                      </v-card>
                  </v-flex>
              </v-layout>
              <div class="text-xs-center">
                  <v-pagination :length="totalPages" v-model="page"></v-pagination>
              </div>
          </v-container>
        </v-content>
        <v-snackbar
                :timeout="3000"
                bottom
                v-model="snackbar">
            Item Deleted!
            <v-btn flat color="green" @click="snackbar = false">X</v-btn>
        </v-snackbar>
    </v-app>
</template>

<script>
    import axios from 'axios';

  export default {
      data(){
          return {
              mouseOver: 0,
              products: [],
              page: 1,
              pageLength: 8,
              snackbar: false,
              searchedText: ''
          }
      },
      methods:{
          cardMedia(product){
              return (this.mouseOver === product.id)
                  ? ((product.images[1] !== undefined && product.images[1] !== '')
                        ? product.images[1].src
                        : product.images[0].src)
                  : product.images[0].src;
          },
          removeItem(product){
              for (let i = 0; i < this.products.length; i++) {
                  let prodToDel = this.products[i];
                  if(prodToDel.id === product.id) {
                      this.products.splice(i, 1);
                      break;
                  }
              }
              this.snackbar = true;
          },
          /**
           * Find a given value for a given obj with requested key
           *
           * @param obj
           * @param key
           * @param val
           * @return {*}
           */
          findObjectByKeyValue: function(obj, key, val) {
              for(let x = 0; x < obj.length; x++)
              {
                  if(obj[x][key] === val)
                      return obj[x];
              }
          }
      },
      computed:{
          totalPages(){
              return Math.ceil(this.products.length/this.pageLength);
          },
          offset(){
              return ((this.page - 1) * this.pageLength);
          },
          limit(){
              return this.page * this.pageLength;
          },
          displayProducts(){
              let products = [];

              if(this.searchedText !== '')
                  for (let i = 0; i < this.products.length; i++){
                      let product = this.products[i];
                      if(product.name.search(new RegExp(this.searchedText, "i")) >= 0)
                          products.push(product);
                  }
              else
                  products = this.products;

              if(products.length === 0)
                  products.push({
                      "id": 1,
                      "name": "No product found",
                      "price": 0.00,
                      "discount_price": 0,
                      "discount_text": 0,
                      "images": [
                          {
                              "src": "public/images/no.png"
                          }
                      ]
                  });

              return products.slice(this.offset, this.limit);
          },
          finalAmount(){
              let amount = 0;

              for (let i = 0; i < this.products.length; i++){
                  let product = this.products[i];

                  if(product.discount_price > 0)
                      amount+=product.discount_price;
                  else
                      amount+=product.price;
              }

              return amount.toFixed(2);
          }
      },
      mounted(){
          let self = this;
          axios.get(
              'public/data.json'
          ).then(function (response) {
              self.products = response.data;
              self.products.forEach(function (data) {
                  self.mouseOver[data.id] = false;
              });
          }).catch(function (error) {
              console.log(error);
          });
      }
  }
</script>
