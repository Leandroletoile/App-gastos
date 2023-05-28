<script setup>
import { reactive, ref, watch } from "vue";
import Swal from "sweetalert2";

const currentDate = ref(getFormattedDate(new Date()));
const formattedDate = ref(formatDate(currentDate.value));
const title = ref("Colocar título evento");
const textColor = ref("#53b89b");

function getFormattedDate(date) {
  const day = String(date.getDate()).padStart(2, "0");
  const month = String(date.getMonth() + 1).padStart(2, "0");
  const year = date.getFullYear();
  return `${year}-${month}-${day}`;
}

function formatDate(dateString) {
  const parts = dateString.split("-");
  const day = parts[2];
  const month = parts[1];
  const year = parts[0];
  return `${day}/${month}/${year}`;
}

const editTitle = async () => {
  const { value } = await Swal.fire({
    title: "Nombre evento",
    input: "text",
    buttonsStyling: false,
    customClass: {
      confirmButton: "btn btn-info",
    },
  });
  if (value) {
    title.value = value;
    textColor.value = "#000000";
  }
};

watch(currentDate, (newDate) => {
  formattedDate.value = formatDate(newDate);
});
</script>


<template>
  <div class="container_header animate__animated animate__zoomIn">
    <h1 class="header_title">
      <label for="datepicker" class="calendar_button">
        <div>
          <i class="bi bi-calendar-week-fill button-select"></i>
          <span title="Elegir fecha">( {{ formattedDate }} )</span>
        </div>
        <input
          type="date"
          id="datepicker"
          v-model="currentDate"
          class="my-datepicker"
        />
      </label>
      &nbsp;&nbsp;<span @click="editTitle" title="Click para editar" :style="{ color: textColor }"
        >" {{ title.charAt(0).toUpperCase() + title.slice(1) }} "</span
      >
    </h1>
  </div>
</template>


<style scoped>
.calendar_button {
  position: relative;
  background-color: transparent;
  display: flex;
  align-items: center;
}

.calendar_button div {
  display: flex;
  align-items: center;
}

.calendar_button i {
  margin-right: 20px;
}

.calendar_button span {
  white-space: nowrap;
}

.my-datepicker {
  opacity: 0;
  position: absolute;
  top: 0;
  right: 199px;
  width: 100%;
  height: 100%;
}

.my-datepicker::-webkit-calendar-picker-indicator {
  cursor: pointer;
}

.button-select{
  padding: 5px 8px 5px 8px;
  border: 5px solid #53b89b;
  border-radius: 50%;
}

.header_title {
  font-size: 1.5em;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #cbf0f5;
  margin: 1rem auto 2rem;
  padding: 25px;
  border-radius: 50px;
  border: 5px solid black;
  box-sizing: border-box;
  min-width: 280px;
  max-width: 1200px;
}
.header_title span {
  white-space: nowrap;
}

@media screen and (max-width: 700px) {
  .header_title {
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
    padding: 15px;
  }
}

@media screen and (min-width: 320px) {
  .header_title {
    min-width: 280px;
    max-width: 600px;
  }
}

@media screen and (min-width: 768px) {
  .header_title {
    min-width: 400px;
    max-width: 800px;
  }
}

@media screen and (min-width: 1024px) {
  .header_title {
    min-width: 600px;
    max-width: 1200px;
  }
}

@media screen and (min-width: 280px) and (max-width: 320px) {
  .header_title {
    min-width: 250px;
    max-width: 450px;
  }
}
</style>