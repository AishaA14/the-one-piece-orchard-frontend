<template>
    <div>
        <h1>Log into your account here</h1>
        <div v-if="isLoggedIn">
        <h2>Hello {{ userName }}</h2>
        <button @click="handleLogOut">Log Out</button>
    </div>
    <div v-else>
        <GoogleLogin :callback="callback" />
    </div>
    </div>
</template>

<script>
 import { decodeCredential, googleLogout } from 'vue3-google-login'
 export default {
    name: 'LoginPage',
    data: () => ({
        isInit: false,
        isLoggedIn: false,
        userName: ''
    }),
    // Check if the user is logged in or not, checks if the cookie exists 
    // To hide certain components if the user is logged out
    mounted(){
        if (this.$cookies.isKey('user_session')) {
            this.isLoggedIn = true
            const userData = decodeCredential(this.$cookies.get('user_session'))
            this.userName = userData.given_name
            } else { 
                console.log('no user found')
            }                  
    },
    methods: {
            callback: function (response) {
                this.isLoggedIn = true
                const userData = decodeCredential(response.credential)
                console.log(userData)
                this.userName = userData.given_name
                this.$cookies.set('user_session', response.credential)
                fetch('http://localhost:4000/user/login', {
                    method: 'POST',
                    headers: {
                    'Content-Type': 'application/json',
                    },
                    body: JSON.stringify ({
                        email: userData.email
                })
            })
                .then(() =>{
                    console.log('session saved')
                    // Redirect to the home page after successful login
                 this.$router.push({ name: 'HomePage' })
                })
            },
            handleLogOut: function () {
                googleLogout()
                this.$cookies.remove('user_session')
                this.isLoggedIn = false
                this.$router.push({ name: 'HomePage' })
                }
        }
    }
</script>

<style>
/* component based styling, as opposed to global styling */
.btn {
    font-size: 24px;
}
</style>