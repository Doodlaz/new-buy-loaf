<template lang="pug">

  #app.global-wrap

    main
      .buy-loaf(:class="{'change-mode' : !changeMode}")
        .buy-loaf__head
          p.buy-loaf__title Список покупок
          button.buy-loaf__remove(@click="changeMode = !changeMode" :class="{'remove-show' : changeMode}")
            span
            span
            span
        .buy-loaf__content
          ul.buy-loaf__list(:class="{ 'remove-show' : changeMode}")
            li.buy-loaf__product(v-for="(product, i) in products", v-if="!product.completed", :key="product.id" @click.capture="moveInCompleted(product, i)")
              p {{product.name}}
              p.count {{product.count}}
              button.product-del(@click.capture="removeProducts(product, i)")

          ul.buy-loaf__list.buy-loaf__list_completed(:class="{'remove-show' : changeMode}")
            li.buy-loaf__product(v-for="(product, i) in products", v-if="product.completed", :key="product.id" @click.capture="moveInCompleted(product, i)")
              p {{product.name}}
              p.count {{product.count}}
              button.product-del(@click.capture="removeProducts(product, i)")
        .btn-group
          .wrap-input
            input#input-product(type="text" v-model="newProduct.name" @keyup.13="focusInCount" placeholder="Продукт" value="")
            input#input-count(type="text" v-model="newProduct.count" @keyup.13="addProducts" placeholder="123")
          input(type="button" @click="addProducts" value="Add", v-if="!changeProduct")
          input(type="button" @click="updateProducts" value="update", v-if="changeProduct")


</template>

<script>
  let config = {
    apiKey: 'AIzaSyASsvJSdkgd9LwC-KY5AZYWXK5OkZOCsw8',
    authDomain: 'project-vue-ec269.firebaseapp.com',
    databaseURL: 'https://project-vue-ec269.firebaseio.com',
    projectId: 'project-vue-ec269',
    storageBucket: 'project-vue-ec269.appspot.com',
    messagingSenderId: '896100021661'
  };
  let firebaseApp = firebase.initializeApp(config);

  import firebase from 'firebase';

  let productsRef = firebaseApp.firestore().collection('products');

  import appMain from './components/main'

  export default {
    name: 'App',
    components: {
      'app-main': appMain
    },
    data() {
      return {
        index: '',
        id: '',
        changeMode: false,
        changeProduct: false,
        products: [],
        newProduct: {
          name: '',
          count: '',
          completed: false
        }
      }

    },

    created() {
      this.getProducts();
      this.watching()
    },

    computed: {

    },

    methods: {

      updateProducts: function () {


        productsRef.add(this.newProduct).then(response => {
          productsRef.doc(this.id).delete();
          this.products.splice(this.index, 1);
          this.products.push({
            name: this.newProduct.name,
            count: this.newProduct.count,
            id:response.id,
            completed: false
          });
          this.sortArray(this.products);
          productsRef.doc(response.id).update({id:response.id});
          this.newProduct.name = '';
          this.newProduct.count = '';
          document.getElementById('input-product').focus();
        });

        this.changeProduct = !this.changeProduct;

      },
      addProducts: function () {

        productsRef.add(this.newProduct).then(response => {
          this.products.push({
            name: this.newProduct.name,
            count: this.newProduct.count,
            id:response.id,
            completed: false
          });
          this.sortArray(this.products);
          productsRef.doc(response.id).update({id:response.id});
          this.newProduct.name = '';
          this.newProduct.count = '';
          document.getElementById('input-product').focus();
        });


      },

      moveInCompleted: function (product, i) {
        if(!this.changeMode){
          productsRef.doc(product.id).update({completed:!product.completed});
          product.completed = !product.completed;
        }else {
          this.newProduct.name = product.name;
          this.newProduct.count = product.count;
          this.index = i;
          this.id = product.id;
          this.changeProduct = !this.changeProduct;
          document.getElementById('input-product').focus();
        }

      },

      removeProducts: function (product, i) {
        productsRef.doc(product.id).delete();
        this.newProduct.name = '';
        this.newProduct.count = '';
        this.products.splice(i, 1);
      },

      getProducts: function () {
        productsRef.get().then(response => {
          let array = [];
          response.forEach(function(doc) {
            array.push(doc.data());

          });
          this.products = array;

          this.sortArray(this.products);

        });

        console.log('getProducts');
      },
      focusInCount : function () {
        document.getElementById('input-count').focus();
      },
      watching: function () {
        setInterval(response => {
          this.getProducts();
        },10000);
      },
      sortArray: function (array) {
        array.sort(function (a,b) {
          if (a.name < b.name)
            return -1;
          if (a.name > b.name)
            return 1;
          return 0;
        });
      }
    },
    mounted() {

    }
  }

</script>

<style lang="scss">
  body{
    background: lightblue;
    font-family: 'Tahoma', sans-serif;
    font-size: 14px;
    color: #181717;
    margin: 0;
    box-sizing: border-box;
  }
  button{
    outline: none;
    cursor: pointer;
  }
  input::-webkit-input-placeholder { /* Chrome/Opera/Safari */
    color: #cccccc;
  }
  input::-moz-placeholder { /* Firefox 19+ */
    color: #cccccc;
  }
  input:-ms-input-placeholder { /* IE 10+ */
    color: #cccccc;
  }
  input:-moz-placeholder { /* Firefox 18- */
    color: #cccccc;
  }
  #app{
    max-width: 100%;
    margin: 0 auto;
  }
  .buy-loaf{
    overflow: hidden;
    max-width: 576px;
    margin: auto;
    height: 100vh;
    max-height: 800px;
    position: relative;
    &__head{
      background: #4471ff;
      color: #ffffff;
      font-size: 18px;
      padding: 13px 16px;
      box-shadow: 0 3px 10px 0 #898989;
      position: relative;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    &__content{
      height: 100%;
      display: flex;
      flex-direction: column;
      border-left: 1px solid;
      border-right: 1px solid;
      max-height: calc(100% - 88px);
      transition: .3s;
      overflow-y: auto;
      overflow-x: hidden;
      .change-mode &{
        max-height: calc(100% - 48px);
      }
    }
    &__list{
      list-style: none;
      margin: 0;
      background: #f8f8f8;
      padding: 0 16px;
      flex: 0 1 100%;
      &_completed{
        background: #464646;
        color: #737373;
        border-top: 1px dashed #f8f8f8;
        li{
          border-top: 1px solid #f8f8f8;
        }
      }
    }
    &__product{
      padding: 13px 0;
      border-top: 1px solid #181717;
      cursor: pointer;
      line-height: 1;
      display: flex;
      justify-content: space-between;
      flex-wrap: wrap;
      position: relative;
      transition: .3s;
      &:first-of-type{
        border-top: none;
      }
      .remove-show &{
        padding-right: 40px;
      }
    }
    &__remove{
      position: absolute;
      right: 16px;
      border: none;
      background: none;
      top: 0;
      bottom: 0;
      margin: auto;
      transition: .3s;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      width: 26px;
      height: 26px;
      padding: 0;
      span{
        display: block;
        width: 26px;
        height: 2px;
        background: #fff;
        margin: 2px;
      }

    }
  }

  .product-del{
    position: absolute;
    right: -60px;
    border: none;
    background: #ff5d54;
    border-radius: 100%;
    height: 26px;
    width: 26px;
    top: 0;
    bottom: 0;
    margin: auto;
    transition: .3s;
    .remove-show &{
      transition: .3s;
      right: 0;
    }
    &:after,
    &:before{
      content: "";
      width: 14px;
      height: 2px;
      background: #fff;
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      margin: auto;
    }
    &:after{
      transform: rotate(45deg);
    }
    &:before{
      transform: rotate(-45deg);
    }
  }

  p{
    margin: 0;
  }
  .btn-group{
    display: flex;
    flex-wrap: wrap;
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    transition: .3s;
    .change-mode &{
      bottom: -40px;
    }
  }
  .btn-group input{
    border: 1px solid #181717;
    height: 40px;
    box-sizing: border-box;
    outline: none;
  }
  .wrap-input{
    width: calc(100% - 80px);
    display: flex;
    position: relative;
  }
  .wrap-input:before{
    content: ":";
    position: absolute;
    line-height: 38px;
    left: 70%;
  }
  .wrap-input input{
    width: 70%;
    border-right: none;
  }
  .wrap-input input:last-of-type{
    border-left: none;
    width: 30%;
  }
  .btn-group input[type='text']{
    padding: 0 10px;
    border-right: none;
    background: #f8f8f8;
  }
  .btn-group input[type='button']{
    width: 80px;
    cursor: pointer;
    background: #4471ff;
    color: #fff;
    transition: .3s;
  }
  .btn-group input[type='button']:hover{
    opacity: .8;
  }
  .count{
    max-width: 60px;
    text-overflow: ellipsis;
    overflow: hidden;
  }


</style>
