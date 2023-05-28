<script setup>
import { data_users } from "./users_data";
import { pastelColors } from "./random_colors";
import Swal from "sweetalert2";
import { reactive, ref } from "vue";
import { card_data } from "./card_data";

const draggingUser = ref(false);
const cardData = reactive(card_data);

const endDrag = () => {
  draggingUser.value = false;
};

const startDrag = (evt, user) => {
  draggingUser.value = true;
  evt.dataTransfer.setData("text/plain", JSON.stringify({ user: user }));
};

function getRandomPastelColor() {
  const randomIndex = Math.floor(Math.random() * pastelColors.length);
  return pastelColors[randomIndex];
}

const onAddUser = async () => {
  const { value: title } = await Swal.fire({
    title: "Nombre usuario",
    input: "text",
    buttonsStyling: false,
    customClass: {
      confirmButton: "btn btn-info",
    },
  });

  if (title) {
    const existingUser = data_users.find(
      (user) => user.name.toLowerCase() === title.toLowerCase()
    );

    if (existingUser) {
      Swal.fire({
        icon: "error",
        title: `"${title.charAt(0).toUpperCase() + title.slice(1)}" ya existe`,
        showConfirmButton: false,
        timer: 1500,
      });
    } else {
      data_users.push({
        name: title,
        color: getRandomPastelColor(),
        total_pay: null,
        total_collect: null,
        total: null,
        id: crypto.randomUUID(),
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
        title: `"${title.charAt(0).toUpperCase() + title.slice(1)}" agregado`,
      });
    }
  }
};

const onDrop = async (evt) => {
  evt.preventDefault();
  const data = evt.dataTransfer.getData("text/plain");
  const { user } = JSON.parse(data);
  await deleteUser(user);
};

const calculateUserTotalsByCard = async (card_data, data_users) => {
  if (!card_data || !data_users) {
    return;
  }
  data_users.forEach((user) => {
    user.userTotalByCard = 0;
  });

  if (Array.isArray(card_data)) {
    card_data.forEach((card) => {
      if (card.users && card.users.length > 0) {
        card.users.forEach((user) => {
          const totalByUser = card.total / card.users.length;
          const userData = data_users.find(
            (dataUser) => dataUser.id === user.id
          );

          if (userData) {
            userData.userTotalByCard += totalByUser;
          }
        });
      }
    });
  }

  data_users.forEach((user) => {
    user.total_pay = user.userTotalByCard;
    user.total = (user.total_collect || 0) + (user.total_pay || 0);
  });
};

const deleteUser = async (user) => {
  if (!user) {
    console.log("Usuario no definido");
    return;
  }

  const deletedUser = { ...user };
  const deletedUserId = deletedUser.id;

  const userIndex = data_users.findIndex((u) => u.id === deletedUserId);
  if (userIndex !== -1) {
    const deletedUserName = data_users[userIndex].name;
    data_users.splice(userIndex, 1);
  } else {
    return;
  }

  for (const card of card_data) {
    if (card.users) {
      const userIndex = card.users.findIndex((u) => u.id === deletedUserId);
      if (userIndex !== -1) {
        card.users.splice(userIndex, 1);
      }
    }

    if (card.orders_data) {
      const filteredOrdersData = card.orders_data.filter((order) => {
        if (order.name_user === deletedUser.name) {
          return false;
        }
        return true;
      });
      card.orders_data = filteredOrdersData;
    }

    if (!card.total || card.orders_data.length === 0) {
      card.users = [];
      card.total = 0;
    }

    let cardTotal = 0;
    for (const order of card.orders_data) {
      cardTotal += order.total;
    }
    card.total = cardTotal;

    const remainingUsers = card.users ? card.users.length : 0;
    const totalPerUser = remainingUsers > 0 ? card.total / remainingUsers : 0;
    if (card.users) {
      card.users.forEach((u) => {
        u.total_pay = totalPerUser;
        u.total = u.total_collect + u.total_pay;
      });
    }

    if (card.users.length === 0 || card.orders_data.length === 0) {
      card.total = 0;
    } else {
      const remainingUsersTotal = card.users.reduce(
        (total, u) => total + u.total_pay,
        0
      );
      card.total = remainingUsersTotal;
    }
  }

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
    icon: "warning",
    title: `"${
      user.name.charAt(0).toUpperCase() + user.name.slice(1)
    }" eliminado`,
  });

  if (card_data && card_data.length > 0) {
    await calculateUserTotalsByCard(card_data, data_users);
  }
};
</script>

<template>
  <div>
    <div
      v-if="!draggingUser"
      title="Añadir usuario"
      class="buttonAddUser animate__animated animate__zoomIn"
    >
      <i @click="onAddUser" type="button" class="bi bi-person-fill-add"></i>
    </div>

    <div v-if="draggingUser" title="Borrar usuario" class="buttonDelUser">
      <i
        @drop="onDrop"
        @dragover.prevent
        @dragenter.prevent
        class="bi bi-trash3-fill"
      ></i>
    </div>

    <div class="container_users">
      <div
        class="animate__animated animate__bounce"
        v-for="user in data_users"
        :key="user.id"
      >
        <div
          draggable="true"
          @dragstart="startDrag($event, user)"
          @dragend="endDrag"
          :style="{ backgroundColor: user.color }"
          class="item_user"
          title="Arrastra usuario al gasto"
        >
          <span class="input-container">
            <input
              :style="{ color: user.color }"
              @input="(e) => (user.color = e.target.value)"
              type="color"
              class="form-control-color colorPicker custom-icon"
              id="exampleColorInput"
              :value="user.color"
              title="Elegir color"
            />
          </span>
          &nbsp;&nbsp;{{
            user?.name.charAt(0).toUpperCase() + user?.name.slice(1)
          }}
          <span>
            <i class="bi bi-arrows-move"></i>
          </span>
        </div>
        <div title="Total gastado" class="total_collect">
          $
          {{
            Math.ceil(user?.total_collect).toLocaleString("es-ES", {
              useGrouping: true,
              minimumFractionDigits: 0,
            })
          }}
        </div>
        <div title="Total consumido" class="total_pay">
          $
          {{
            Math.ceil(user?.total_pay).toLocaleString("es-ES", {
              useGrouping: true,
              minimumFractionDigits: 0,
            })
          }}
        </div>
        <div
          :style="{
            color:
              user.total > 0 ? 'green' : user.total === 0 ? 'black' : 'red',
          }"
          class="total"
          title="Rojo: total cobrar - Verde: total pagar"
        >
          $
          {{
            Math.ceil(user?.total).toLocaleString("es-ES", {
              useGrouping: true,
              minimumFractionDigits: 0,
            })
          }}
        </div>
      </div>
    </div>
  </div>
</template>


<style scoped>
.container_all {
  display: flex;
  flex-direction: column;
}

.buttonAddUser i {
  font-size: 3.4em;
  display: flex;
  justify-content: center;
  width: fit-content;
  margin: 0 auto;
  border: 10px solid #01a5e4;
  padding: 8px;
  border-radius: 50%;
  flex-grow: 0;
}
.buttonDelUser i {
  font-size: 3.4em;
  display: flex;
  justify-content: center;
  width: fit-content;
  margin: 0 auto;
  border: 10px solid #e83c50;
  padding: 8px;
  border-radius: 50%;
  flex-grow: 0;
}

.container_users {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  margin: 2rem 0;
  gap: 15px;
  color: white;
}

.item_user {
  display: flex;
  font-size: x-large;
  margin-bottom: 15px;
  padding: 5px 10px 5px 0px;
  border-radius: 25px;
  border: 2px dotted white;
  justify-content: center;
  align-items: center;
  align-content: center;
  cursor: grab;
}

.item_user span {
  margin-left: 13px;
}

.total_collect,
.total_pay,
.total {
  color: black;
  display: flex;
  justify-content: center;
  margin: 1rem 0;
  padding: 2px 8px;
  border: 2px solid black;
  border-radius: 5px;
  cursor: pointer;
}

.total_collect {
  font-size: 18px;
  background-color: pink;
}

.total_pay {
  font-size: 18px;
  background-color: #e6ecf5;
}

.total {
  font-size: 18px;
  background-color: white;
  padding: 10px 0 10px 0;
}

.colorPicker {
  transform: translateY(3px);
  border-radius: 250px;
  width: 30px;
  height: 25px;
  padding: 0;
  border: none;
  border: 8px solid white;
}

.form-control-color::-webkit-color-swatch {
  border: none;
  border-radius: 250px;
  width: 30px;
  height: 15px;
}
</style>
