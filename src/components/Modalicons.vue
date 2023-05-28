<script setup>
import { reactive } from "vue";
import { iconsObject } from "./icons";
import { card_data } from "./card_data";

const props = defineProps({
  data: {
    type: Object,
    required: true,
  },
  showModal: {
    type: Boolean,
    required: true,
  },
});

const onSelectIcon = (e, data) => {
  data.iconSelected = e.target.classList[1];
};
</script>

<template>
  <div>
    <div type="button" class="header_msg button" @click="showModal = true">
      <i :class="data.iconSelected"></i>
    </div>

    <div class="modal-mask" v-show="showModal" @click.self="showModal = false">
      <div
        class="modal-wrapper d-flex align-items-center justify-content-center"
      >
        <div class="modal-dialog">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="exampleModalLabel">
                Seleccionar icono
              </h5>
            </div>
            <div class="modal-body text-dark">
              <div class="icon-list">
                <i
                  v-for="icon in iconsObject"
                  :key="icon.id"
                  type="button"
                  :class="icon"
                  @click="
                    (e) => {
                      onSelectIcon(e, data);
                      showModal = false;
                    }
                  "
                >
                </i>
              </div>
            </div>
            <div class="modal-footer">
              <button
                type="button"
                class="btn btn-info"
                @click="showModal = false"
              >
                OK
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<style scoped>
.button {
  display: inline-block;
  border: 2px solid white;
  border-radius: 50%;
  background-color: black;
  padding: 2px 10px 2px 10px;
  transition: all 0.3s ease;
  cursor: pointer;
  margin-bottom: 20px;
}

.button:hover {
  background-color: white;
  color: black;
}

.icon-list {
  color: black;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  gap: 10px;
  max-height: 70vh;
  overflow-y: auto;
}
.icon-list i {
  font-size: 1.3em;
}

.modal-title {
  color: #545454;
  font-size: 1em;
  letter-spacing: -1px;
}

.modal-mask {
  position: fixed;
  z-index: 9998;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: table;
  transition: opacity 1.5s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}

.modal-dialog {
  display: inline-block;
  margin: 1.75rem;
  vertical-align: middle;
  max-width: 500px;
  width: 100%;
  position: relative;
  z-index: 9999;
}

.modal-content {
  background-color: #fff;
  border-radius: 0.3rem;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.2);
  margin: 0 auto;
  padding: 1.5rem;
  position: relative;
  transition: opacity 1.5s ease;
}

.modal-enter {
  opacity: 0;
}

.modal-enter-active {
  opacity: 1;
  transition: opacity 1.5s ease;
}

.modal-leave {
  opacity: 1;
}

.modal-leave-active {
  opacity: 0;
  transition: opacity 1.5s ease;
}
</style>