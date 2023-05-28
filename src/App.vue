<script setup>
import Header from "./components/Header.vue";
import CardBills from "./components/CardBills.vue";
import UsersItems from "./components/UsersItems.vue";
</script>

<template>
  <div class="body">
    <Header />
    <CardBills />
    <UsersItems />
  </div>
</template>

<style scoped>

.body {
  display: flex;
  flex-direction: column;
  align-items: center;
  background-image: url("./assets/descarga.png");
  background-attachment: fixed;
  background-position: center;
  background-repeat: repeat;
  background-size: contain;
  padding-top: 1rem;
  min-height: 100vh;
  min-width: 100vw;
}
body.modal-open {
  overflow: hidden;
  height: 100vh;
}

</style>
