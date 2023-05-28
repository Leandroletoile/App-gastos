<script setup>
import Modalicons from "./Modalicons.vue";
import Swal from "sweetalert2";
import { data_users } from "./users_data";
import { card_data } from "./card_data";
import { reactive } from "vue";
import { iconsObject } from "./icons";
import { ref } from "vue";

const showModal = ref(false);

const props = defineProps({
  data: {
    type: Object,
    required: true,
  },
});

const onNewMsg = async () => {
  const { value: text } = await Swal.fire({
    input: "textarea",
    title: "Mensaje",
    inputPlaceholder: "Escriba su mensaje...",
    showCancelButton: true,
    buttonsStyling: false,
    customClass: {
      confirmButton: "btn btn-info",
      cancelButton: "btn btn-secondary",
    },
  });
  if (text) {
    Swal.fire({
      text: text,
      confirmButtonColor: "#0dcaf0",
    });
  }
};

const onHandleAdd = async (onData) => {
  const cardData = card_data.find((data) => data.id === onData.id);

  const Nombre = data_users.map((user) => user.name);

  const nombreOptions = Nombre.map((nombre) => {
    const nombreCapitalized = nombre.charAt(0).toUpperCase() + nombre.slice(1);
    return nombreCapitalized;
  });

  const { value: formValues } = await Swal.fire({
    html: `
      <input id="swal-input2" placeholder="Total $$$" class="swal2-input custom-input" type="text" inputmode="numeric" pattern="[0-9]*">
    `,
    title: "Ingrese nuevo gasto",
    input: "select",
    inputPlaceholder: "Usuario",
    inputOptions: nombreOptions,
    showCancelButton: true,
    buttonsStyling: false,
    customClass: {
      confirmButton: "btn btn-info",
      cancelButton: "btn btn-secondary",
    },
    preConfirm: () => {
      return [document.getElementById("swal-input2").value];
    },
    inputValidator: (value) => {
      return new Promise((resolve) => {
        const value2 = document.getElementById("swal-input2").value;

        if (!value2.trim()) {
          resolve("Debe ingresar un monto");
          return;
        }

        if (!/^\d+$/.test(value2)) {
          resolve("El monto debe ser un número entero");
          return;
        }

        if (value.length <= 0) {
          resolve("Debe ingresar un nombre");
          return;
        }

        const inputField = document.getElementById("swal-input2");
        inputField.addEventListener("input", formatInputValue);

        function formatInputValue(event) {
          const { value } = event.target;
          const formattedValue = addThousandsSeparator(value);
          event.target.value = formattedValue;
        }

        const nombreUsuario = Nombre[value];

        const userFinded = data_users.find(
          (user) => user.name === nombreUsuario
        );
        const userExists = cardData.users.some(
          (user) => user.name === nombreUsuario
        );

        if (!userExists) {
          cardData.users.push(userFinded);
        }

        cardData.orders_data.push({
          id: crypto.randomUUID(),
          name_user: nombreUsuario,
          total: Number(value2),
        });

        let total_cost = 0;
        for (let i = 0; i < cardData.orders_data.length; i++) {
          total_cost += cardData.orders_data[i].total;
        }
        cardData.total = Number(total_cost);

        for (let persona of cardData.users) {
          const total_on_add = isNaN(
            Math.ceil(cardData.total[cardData.total.length - 1]?.total)
          )
            ? 0
            : Math.ceil(cardData.total[cardData.total.length - 1]?.total);
          const users_on_add = cardData.users.length;
          const res = isNaN(total_on_add / users_on_add)
            ? null
            : total_on_add / users_on_add;

          let nameToSearch = persona.name;
          let personSearch = data_users.find((p) => p.name === nameToSearch);

          personSearch.total_pay = res;
          personSearch.total =
            personSearch.total_collect + personSearch.total_pay;
        }
        data_users[value].total_collect -= value2;

        calculateUserTotalsByCard(card_data, data_users);

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
          title: "Gasto por usuario",
        });
      });
    },
    didOpen: () => {
      Swal.getPopup().querySelector("#swal-input2").focus();
      // const swalContainer = Swal.getContent().querySelector(
      //   ".swal2-html-container"
      // );
      // swalContainer.style.overflow = "hidden";
      // swalContainer.style.height = "100%";
      // swalContainer.style.display = "flex";
      // swalContainer.style.flexDirection = "column";
      // const isScreenWidthLessThan437 = window.innerWidth < 437;
      // const swalInput2 = Swal.getContent().querySelector("#swal-input2");

      // if (isScreenWidthLessThan437) {
      //   swalInput2.classList.add("hide-scrollbar");
      // }
    },
  });
};

const calculateUserTotalsByCard = (card_data, data_users) => {
  data_users.forEach((user) => {
    user.userTotalByCard = 0;
  });

  card_data.forEach((card) => {
    card.users.forEach((user) => {
      const totalByUser = card.total / card.users.length;
      const userData = data_users.find(
        (dataUser) => dataUser.name === user.name
      );
      userData.userTotalByCard += totalByUser;
    });
  });

  data_users.forEach((user) => {
    user.total_pay = user.userTotalByCard;
    user.total = (user.total_collect || 0) + (user.total_pay || 0);
  });
};
</script>




<template>
  <div class="header_card">
    <!-- BOTON DE ICONOS -->

    <Modalicons
      class="d-flex justify-content-center"
      :showModal="showModal"
      :data="data"
      title="Elegir ícono"
    />

    <!-- BOTON DE ICONOS -->

    <!-- ICONO MENSAJE HEADER -->
    <div class="header_align_center">
      <div title="Añadir mensaje" class="button">
        <div class="header_icon">
          <svg
            @click="onNewMsg"
            xmlns="http://www.w3.org/2000/svg"
            width="40"
            height="40"
            fill="currentColor"
            class="bi bi-chat-text-fill"
            viewBox="0 0 16 16"
          >
            <path
              d="M10.067.87a2.89 2.89 0 0 0-4.134 0l-.622.638-.89-.011a2.89 2.89 0 0 0-2.924 2.924l.01.89-.636.622a2.89 2.89 0 0 0 0 4.134l.637.622-.011.89a2.89 2.89 0 0 0 2.924 2.924l.89-.01.622.636a2.89 2.89 0 0 0 4.134 0l.622-.637.89.011a2.89 2.89 0 0 0 2.924-2.924l-.01-.89.636-.622a2.89 2.89 0 0 0 0-4.134l-.637-.622.011-.89a2.89 2.89 0 0 0-2.924-2.924l-.89.01-.622-.636zM6 7.5h4a.5.5 0 0 1 0 1H6a.5.5 0 0 1 0-1z"
            />
          </svg>
        </div>
      </div>
      <!-- ICONO MENSAJE HEADER -->

      <div class="header_title">
        {{ data.title.charAt(0).toUpperCase() + data.title.slice(1) }}
      </div>

      <!-- BOTON ADD + -->
      <div title="Añadir gasto individual" class="button">
        <div class="icon_plus">
          <svg
            @click="onHandleAdd(data)"
            xmlns="http://www.w3.org/2000/svg"
            width="40"
            height="40"
            fill="currentColor"
            class="bi bi-patch-plus-fill"
            viewBox="0 0 16 16"
          >
            <path
              d="M10.067.87a2.89 2.89 0 0 0-4.134 0l-.622.638-.89-.011a2.89 2.89 0 0 0-2.924 2.924l.01.89-.636.622a2.89 2.89 0 0 0 0 4.134l.637.622-.011.89a2.89 2.89 0 0 0 2.924 2.924l.89-.01.622.636a2.89 2.89 0 0 0 4.134 0l.622-.637.89.011a2.89 2.89 0 0 0 2.924-2.924l-.01-.89.636-.622a2.89 2.89 0 0 0 0-4.134l-.637-.622.011-.89a2.89 2.89 0 0 0-2.924-2.924l-.89.01-.622-.636zM8.5 6v1.5H10a.5.5 0 0 1 0 1H8.5V10a.5.5 0 0 1-1 0V8.5H6a.5.5 0 0 1 0-1h1.5V6a.5.5 0 0 1 1 0z"
            />
          </svg>
        </div>
      </div>
      <!-- BOTON ADD + -->
    </div>
  </div>
</template>

<style scoped>
/* .swal2-html-container {
  overflow: hidden !important;
}
.hide-scrollbar {
  overflow-x: hidden;
}
#swal2-html-container::-webkit-scrollbar {
  display: none;
}
.swal2-input::-webkit-scrollbar {
  display: none;
} */

.buttonAddCard i {
  font-size: 2em;
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
}

.header_card {
  display: flex;
  flex-direction: column;
  border: 5px solid #2489ab;
  background-color: black;
  padding: 20px;
  color: white;
  font-size: 2em;
  border-radius: 0 25px 25px 25px;
  box-sizing: border-box;
  border-right: none;
  border-bottom: none;
}
.btn_modal {
  display: flex;
  justify-content: center;
}

.header_align_center {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
}

svg {
  padding: 8px;
}

.button {
  display: inline-block;
  border: 2px solid white;
  border-radius: 50%;
  background-color: black;
  padding: 4px;
  transition: all 0.3s ease;
  padding: 3px;
  cursor: pointer;
}

.button:hover {
  background-color: white;
  color: black;
}

.button svg {
  display: block;
  margin: auto;
}
</style>