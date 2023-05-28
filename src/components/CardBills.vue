<script setup>
import Modalicons from "./Modalicons.vue";
import CardBody from "./CardBody.vue";
import HeaderCard from "./HeaderCard.vue";
import Swal from "sweetalert2";
import { card_data } from "./card_data";

const onAddNewGroupExpense = async () => {
  const { value: title } = await Swal.fire({
    title: "Nombre gasto grupal",
    input: "text",
    inputAttributes: {
      autofocus: true,
    },
    buttonsStyling: false,
    customClass: {
      confirmButton: "btn btn-info",
    },
  });

  if (title) {
    card_data.push({
      id: crypto.randomUUID(),
      title: title,
      iconSelected: "bi bi-images",
      users: [],
      orders_data: [],
      total: null,
    });
    const Toast = Swal.mixin({
      toast: true,
      position: "top-end",
      showConfirmButton: false,
      timer: 3000,
      timerProgressBar: true,
      didOpen: (toast) => {
        toast.addEventListener("mouseenter", Swal.stopTimer);
        toast.addEventListener("mouseleave", Swal.resumeTimer);
      },
    });

    Toast.fire({
      icon: "success",
      title: `"${title.charAt(0).toUpperCase() + title.slice(1)}"`,
    });
  }
};
</script>

<template>
  <div>
    <!-- BOTON AGREGAR NUEVO GASTO GRUPAL -->
    <div class="buttonAddCard">
      <i
        @click="onAddNewGroupExpense"
        title="Añadir gasto grupal"
        type="button"
        class="bi bi-plus-circle-fill animate__animated animate__zoomIn"
      ></i>
    </div>
    <!-- BOTON AGREGAR NUEVO GASTO GRUPAL -->

    <CardBody v-if="card_data.length > 0" />
  </div>
</template>

<style scoped>
.buttonAddCard {
  font-size: 2.5em;
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
}

i {
  border: 10px solid #f200bc;
  padding: 4px 15px 4px 15px;
  border-radius: 50%;
}
</style>

