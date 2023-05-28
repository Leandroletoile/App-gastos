<script setup>
import HeaderCard from "./HeaderCard.vue";
import Swal from "sweetalert2";
import { data_users } from "./users_data";
import { card_data } from "./card_data";

const onDrop = (evt, dest, card) => {
  const { user } = JSON.parse(evt.dataTransfer.getData("text/plain"));

  const cardData = card_data.find((cardItem) => cardItem.id === card.id);

  if (!cardData) {
    console.log("Card not found");
    return;
  }

  if (!cardData.total) {
    Swal.fire({
      icon: "error",
      title: "Debe agregar un gasto primero",
      timer: 3000,
      showConfirmButton: false,
    });
    return;
  }

  const existingUser = cardData.users.find((u) => u.name === user.name);

  if (!existingUser) {
    cardData.users.push(user);
  }

  calculateUserTotalsByCard(card_data, data_users);

  return { card_data, data_users };
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

const onDeleteUser = (e, index, data = {}) => {
  try {
    const userIdToDelete = e.target.textContent.toString();
    const cardIndexToDeleteFrom = card_data.findIndex(
      (card) => card.id === data?.id
    );

    const cardToDeleteFrom = card_data[cardIndexToDeleteFrom];

    const userIndexToRemove = cardToDeleteFrom.users.findIndex(
      (user) => user.name === userIdToDelete
    );

    if (userIndexToRemove === -1) {
      console.log("User not found");
      return;
    }

    cardToDeleteFrom.users.splice(userIndexToRemove, 1);

    calculateUserTotalsByCard(card_data, data_users);
  } catch (error) {
    // Capturar y manejar la excepción sin mostrarla en la consola
  }
};

const formatNumber = (number) => {
  return Math.ceil(number).toLocaleString("es-ES", {
    useGrouping: true,
    minimumFractionDigits: 0,
  });
};
</script>

<template>
  <div>
    <div
      class="container_card"
      v-for="(data, index) in card_data"
      :key="data.id"
    >
      <HeaderCard :data="data" />

      <div
        class="body_card"
        @dblclick="onDeleteUser($event, index, data)"
        @drop="onDrop($event, index, data)"
        @dragover.prevent
        @dragenter.prevent
        style="
          -webkit-user-select: none;
          -moz-user-select: none;
          -ms-user-select: none;
          user-select: none;
        "
      >
        <div v-if="data.orders_data && data.orders_data.length">
          <div class="items" v-for="order in data.orders_data" :key="order.id">
            <div class="item">
              ${{ formatNumber(order.total) }}
              <span>{{
                order.name_user.charAt(0).toUpperCase() +
                order.name_user.slice(1)
              }}</span>
            </div>
          </div>
        </div>

        <div class="total" v-if="data.orders_data && data.orders_data.length">
          ${{ formatNumber(data.total) }}
        </div>

        <div class="body_users">
          <div
            title="Doble click para eliminar"
            class="item_users element-with-close-cursor"
            v-for="user in data.users"
            @dblclick="onDeleteUser"
            :key="user.id"
            :style="{ backgroundColor: user.color }"
          >
            {{ user.name }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container_card {
  flex-wrap: wrap;
  max-width: 350px;
  min-width: 350px;
}
.body_card {
  margin-bottom: 2.5rem;
  border-radius: 5px;
  padding: 2% 0;
  min-height: 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #d4e3fa;
  font-size: 2.5em;
  box-sizing: border-box;
  text-align: center;
  border-right: 7px solid black;
  border-bottom: 7px solid black;
}
.body_users {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  color: white;
  margin-top: 15px;
}
.item_users {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 0.5em;
  padding: 5px 20px;
  border-radius: 25px;
  cursor: crosshair;
  border: 2px solid white;
  margin: 5px;
}
.items {
  margin: 15px 0;
}

.item span {
  border-radius: 15px;
  padding: 2px 15px;
  background-color: pink;
}
.total {
  background-color: white;
  padding: 5px 20px;
  border-radius: 25px;
  margin-top: 15px;
  border: 2px solid #000;
}
@media screen and (min-width: 320px) {
  .container_card {
    min-width: 280px;
    max-width: 600px;
  }
}

@media screen and (min-width: 768px) {
  .container_card {
    min-width: 400px;
    max-width: 800px;
  }
}

@media screen and (min-width: 1024px) {
  .container_card {
    min-width: 600px;
    max-width: 1200px;
  }
}

@media screen and (min-width: 280px) and (max-width: 540px) {
  .container_card {
    min-width: 280px;
    max-width: 540px;
  }
}
@media screen and (min-width: 2152px) {
  .header_title {
    min-width: 800px;
    max-width: 1600px;
    margin: 2rem auto;
  }
}
</style>