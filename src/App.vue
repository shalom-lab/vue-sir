<template>
    <div id="app">
        <Home />
    </div>
</template>

<script>
import Home from './views/Home.vue'

export default {
    name: 'App',
    components: {
        Home
    }
}
</script>

<style>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    margin: 0;
    padding: 0;
    width: 100%;
    height: 100vh;
}

#app {
    width: 100%;
    height: 100%;
}
</style>