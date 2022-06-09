<script>
export default {
  name: 'App'
}
</script>

<template>
  <div class="_height:100% _display:flex _flex-direction:column">
    <i-nav class="_display:flex _justify-content:space-between">
      <i-nav-item href="https://inkline.io">
        <img height="40" src="@/assets/logo.svg">
        <span class="ml-2">Byteplug JSON Validator</span>
      </i-nav-item>
      <i-nav>
        <i-nav-item href="https://www.byteplug.io/standards/json-validator">Learn</i-nav-item>
        <i-nav-item href="https://www.byteplug.io/standards/json-validator/document">Rules</i-nav-item>
        <i-nav-item href="https://www.byteplug.io/standards/json-validator/toolkit">Toolkit</i-nav-item>
      </i-nav>
    </i-nav>
    <div class="_flex-grow:1">
      <router-view/>
    </div>
    <div class="_display:flex _justify-content:center">
      <div class="_flex-grow:1">
        <svg height="60" width="100%">
          <line x1="6%" y1="48px" x2="94%" y2="48px" style="stroke:#d60000;stroke-width:1" />
        </svg>
      </div>
      <div class="_margin-bottom:2">
        <img src="@/assets/logo.svg" height="60" alt="">
      </div>
      <div class="_flex-grow:1">
        <svg height="60" width="100%">
          <line x1="6%" y1="48px" x2="94%" y2="48px" style="stroke:#d60000;stroke-width:1" />
        </svg>
      </div>
    </div>
  </div>
</template>

<style>
html, body, #app {
  height: 100%;
}
</style>