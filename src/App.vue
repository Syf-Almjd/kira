<template>
  <NaviBar />
  <UserProfile />
  <UsersList msg="Welcome to Your Vue.js App" />
</template>

<script>
import NaviBar from './components/NaviBar.vue'
import UserProfile from './components/UserProfile.vue'
import UsersList from './components/UsersList.vue'

export default {
  name: 'App',
  components: {
    NaviBar,
    UserProfile,
    UsersList
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  text-align: center;
}
</style>
