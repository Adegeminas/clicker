<template>
  	<Clicker difficulty="easy" @success="() => console.log('success')" @fail="() => console.log('fail')"/>
</template>

<script setup lang="ts">
import Clicker from './components/Clicker.vue';
</script>

<style scoped>
</style>
