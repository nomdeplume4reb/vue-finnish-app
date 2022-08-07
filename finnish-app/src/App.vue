<template>
  <div>
    <header id="header" class="header">
      <div class="inner">
          <a href="https://nomdeplume4reb.github.io/portfolio/" class="logo">
            <span class="symbol"><img id="profile-shadow" src="./assets/profile-shadow.png" /></span><span class="title">Home</span>
          </a>
      </div>
    </header>
    <FillInBlank></FillInBlank>
  </div>
</template>

<script>
// import _ from 'lodash';
// import natural from 'natural';

import FillInBlank from './components/FillInBlank.vue'

export default {
  name: 'App',
  components: {
    FillInBlank
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400&display=swap');

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

body {
  font-family: 'Poppins', sans-serif;
  background-image: url("./assets/bg-forest.jpg");
  background-size: cover;
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-opacity: .8;
  position: relative;
}

#header .logo {
  position:fixed;
  top:0;
  left:0;
  color: inherit;
  font-weight: 600;
  letter-spacing: 0.35em;
  margin: 10px 0px 0px 10px;
  text-decoration: none;
  text-transform: uppercase;
  display: inline-block;
}

#header .logo > * {
  display: inline-block;
  vertical-align: middle;
}

#header .logo .symbol {
  margin-right: 0.65em;
}

#header .logo .symbol img {
  display: block;
  width: 2em;
  height: 2em;
}

#header:hover {
  color: #6495ED
}


.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  font-size: 1.3em;
  font-weight: 900;
}
</style>
