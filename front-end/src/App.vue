<template>
  <div>
    <dialog-box />
    <div class="nav navbar navbar-inverse">
      <div class="container">
        <router-link to="/"><div class="navbar-brand">{{title}}</div></router-link>
        <ul class="nav navbar-nav">
          <li v-if="isLoggedin"><router-link to="/user/friends">Friends</router-link></li>
          <li v-if="isLoggedin"><router-link to="/user/friendrequests">Friend requests</router-link></li>
        </ul>
        <ul class="nav navbar-nav navbar-right">
          <li v-if="!isLoggedin"><router-link to="/login">Login</router-link></li>
          <li v-if="!isLoggedin"><router-link to="/signup">Sign up</router-link></li>
          <li v-if="isLoggedin"><router-link to="/user/profile">{{currentUser}}</router-link></li>
          <li v-if="isLoggedin"><a href="#" v-on:click="logout">Logout</a></li>
        </ul>
      </div>
    </div>
    <chat v-if="isLoggedin" :currentUser="currentUser"></chat>
    <router-view :currentUser="currentUser"></router-view>
  </div>
</template>

<script>
import axios from 'axios';
import routes from '../config/routes'
import Chat from './components/chat/Chat'
import DialogBox from './components/common/DialogBox'
axios.defaults.withCredentials = true;

export default {
  name: 'app',
  data () {
    return {
      title: 'Chat application',
      currentUser: '',
      isLoggedin: false
    }
  },
  methods: {
    logout: function() {
      axios.get(routes.apiRoutes.logout(this.currentUser));
      this.currentUser =　'';
      this.isLoggedin = false;
      this.$router.push('/');
    },
    authenticateUser: function() {
      axios.get(routes.apiRoutes.authenticateUser)
        .then(response => {
          if (response.data.success) {
            this.currentUser = response.data.userId;
            this.isLoggedin = true;
          } else {
            // var cookies = document.cookie.split(';');
            // document.cookie = '';
            // for (var i = 0; i < cookies.length; i++) {
            //   var eqPos = cookies[i].lastIndexOf('=');
            //   var name = cookies[i].substring(0, eqPos);
            //   if (name !== 'sessionId' && name !== 'securityToken') {
            //     document.cookie += cookies[i];
            //   }

            // }
            if (this.$router.currentRoute.path != '/'
                && this.$router.currentRoute.path != '/login'
                && this.$router.currentRoute.path != '/signup') {

              this.isLoggedin = false;
              this.$router.push('/login');
            }
          }
        });

    }
  },
  created: function() {
    this.authenticateUser();
  },
  watch: {
    '$route': function() {
      this.authenticateUser();
    }
  },
  components: {
    Chat, DialogBox
  }
}
</script>

<style>
  @import '../node_modules/bootstrap/dist/css/bootstrap.min.css';

  .navbar {
    margin-bottom: 0;
    border-radius: 0;
  }
</style>
