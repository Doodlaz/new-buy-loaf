<template lang="pug">

  #app.global-wrap
    main Test Блок
      form#form(v-on:submit.prevent="addUser")
        input(type="text" v-model="newUser.name" placeholder="Username")
        input(type="email" v-model="newUser.email" placeholder="email@email.com")
        input(type="submit" value="Add User")

      ul
        li(v-for="user in users", class="user", :key="user.id")
          span {{user.name}} - {{user.email}}
          button(v-on:click="removeUser(user.id)") X

      //ul.errors
        li(v-show="!validation.name") Name cannot be empty.
        li(v-show="!validation.email") Please provide a valid email address.

</template>

<script>
  let emailRE = /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/

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

  let usersRef = firebaseApp.firestore().collection('users');

  import appMain from './components/main'

  export default {
    name: 'App',
    components: {
      'app-main': appMain
    },
    data() {
      return {
        users: [],
        newUser: {
          name: '',
          email: ''
        },

      }

    },
//    firebase: {
//      users: usersRef
//    },

    created() {
      this.getItam();
    },

    computed: {
      validation: function () {
        return {
          name: !!this.newUser.name.trim(),
          email: emailRE.test(this.newUser.email)
        }
      },
      isValid: function () {
        let validation = this.validation;
        return Object.keys(validation).every(function (key) {
          return validation[key]
        })
      }
    },
    methods: {
      addUser: function () {
        if (this.isValid) {
            usersRef.add(this.newUser).then(function(docRef) {
            usersRef.doc(docRef.id).update({id:docRef.id})


          });
          this.newUser.name = '';
          this.newUser.email = '';
          this.getItam();
        }
      },
      removeUser: function (id) {
        usersRef.doc(id).delete()
        this.getItam();
      },
      getItam: function () {
        let usersArr = []
        usersRef.get()
            .then(function(querySnapshot) {
              querySnapshot.forEach(function(doc) {
                usersArr.push(doc.data());
              });
            })
            .catch(function(error) {
              console.log("Error getting documents: ", error);
            });

        this.users = usersArr;
      }
    },
    mounted() {}
  }

</script>

<style lang="scss">
  *{
    box-sizing: border-box;
    font-size: 100%;
    outline: none;
    text-decoration: none;
    color: #000;
    margin: 0;
    padding: 0;
    list-style: none;

  }
  a, input, button{
    transition: .3s;
  }

  ::-webkit-input-placeholder {color:#ccc;}
  ::-moz-placeholder          {color:#ccc;}/* Firefox 19+ */
  :-moz-placeholder           {color:#ccc;}/* Firefox 18- */
  :-ms-input-placeholder      {color:#ccc;}

  body{
    background: #f1f1f1;
    margin: 0;
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    font-size: 14px;

  }
  ul {
    padding: 0;
  }
  #form{
    margin: 20px 0;
  }
  .user {
    line-height: 30px;
    padding: 10px;
    border-top: 1px solid #eee;
    overflow: hidden;
    transition: all .25s ease;
  }

  .user:last-child {
    border-bottom: 1px solid #eee;
  }

  .v-enter, .v-leave-active {
    height: 0;
    padding-top: 0;
    padding-bottom: 0;
    border-top-width: 0;
    border-bottom-width: 0;
  }

  .errors {
    color: #f00;
  }
</style>
