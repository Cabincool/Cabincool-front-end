<template>
  <button
  v-if="notLogin"
    @click="signin"
    class="login-btn"
  >
    使用<img
      class="login-btn__logo"
      src="../assets/google-login.png"
    >登入</button>
    <img v-bind:src="imgSrc" class="profile_pic" v-else>
</template>

<script>
// import Vuex from 'vuex'
import firebase from 'firebase'
import { url } from '../url'
export default {
  data: () => ({
    clientId: '149578749039-8ki9dmmfnod66fl59hd6mduedr3rvre3.apps.googleusercontent.com',
    token: '',
    notLogin: true,
    imgSrc: localStorage.getItem('avator_url')
  }),
  watch: {
    $route: function () {
      if (localStorage.getItem('token') !== '') {
        console.log(localStorage.getItem('token'))
        this.notLogin = false
      }
    }
  },
  methods: {
    signin () {
      var provider = new firebase.auth.GoogleAuthProvider()
      firebase.auth().signInWithPopup(provider).then(function (result) {
        console.log(result.credential)
        if (result.credential) {
          firebase.auth().currentUser.getIdToken(/* forceRefresh */ true).then(function (idToken) {
            fetch(url + 'api/login', {
              method: 'post',
              body: JSON.stringify({
                'idToken': idToken
              }),
              headers: new Headers({
                'Content-Type': 'application/json'
              })
            }).then(data =>
              data.json().then(data => {
                localStorage.setItem('token', data.access_token)
                fetch(url + 'api/users/info', {
                  method: 'get',
                  headers: new Headers({
                    'Authorization': 'Bearer ' + data.access_token,
                    'Content-Type': 'application/json'
                  })
                }).then(data => {
                  data.json().then(
                    data => {
                      console.log(data)
                      localStorage.setItem('email', data.email)
                      localStorage.setItem('userName', data.name)
                      localStorage.setItem('avator_url', data.avator_url)
                      localStorage.setItem('uid', data.uid)
                    }
                  )
                }
                )
              })
            )
          }).catch(function (error) {
            console.log(error)
          })
        }
      })
      if (localStorage.getItem('token') !== '') {
        console.log(localStorage.getItem('token'))
        this.notLogin = false
      }
    }
  }
}

// firebase.auth().signInWithRedirect(provider)
// firebase.auth().currentUser.getIdToken().then(function (idToken) {
//   console.log(idToken)
// })
//   } else {
//     console.log(this.idToken)
//   }
// }).catch(function (error) {
//   alert(error.code)
// })
//   }
</script>

<style lang="scss">
.profile_pic{
  border-radius: 50%;
  width: 30px;
  height: 30px;
}
.login-btn {
  display: flex;
  align-items: center;
  margin-left: 22px;
  padding: 5px 7px;
  border: 2px solid #435058;
  background: white;
  z-index: 1;
  outline: none;
  transition: 0.2s;
  color: black !important;
  &:hover {
    background: #848c8e;
    border: 2px solid #848c8e;
    color: white;
  }
  &:active {
    background: #435058;
    border: 2px solid #435058;
    color: white;
  }
  &__logo {
    width: 15px;
    padding: 0 8px;
  }
}
</style>
