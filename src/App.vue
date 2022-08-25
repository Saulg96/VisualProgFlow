<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'

</script>

<template>
  <header>
    <main class="title">
      <span class="title-first">Visual</span><span class="title-second">Prog</span><span class="title-third">Flow</span>
    </main>
    <div class="wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>

  <RouterView />
</template>

<style scoped>
header {
  place-items: center;
  line-height: 1.5;
  max-height: 100vh;
}

.title{
  text-align: center;
  font-family: Montserrat;
  font-size: 50px;
  font-weight: 700;
  font-style: normal;
  font-stretch: normal;
  line-height: normal;
  letter-spacing: 0.3em;
}

.title-first{
  color: red;
}
.title-second{
  color:blue;
}

.title-third{
  color: green;
}

nav {
  width: 100%;
  font-size: 25px;
  font-family: Montserrat;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color:hsla(160, 100%, 37%, 1);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid black;
}

nav a:first-of-type {
  border: 0;
}

</style>
