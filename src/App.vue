<script setup lang="ts">
import { ref } from "vue";
import EcSwiper from "./components/EcSwiper.vue";
// import {EcSwiper} from "@ec50n9/ec-swiper";
// import "@ec50n9/ec-swiper/style.css";

const dataList = [
  { name: "a", age: 19, background: "red" },
  { name: "b", age: 20, background: "blue" },
  { name: "c", age: 21, background: "green" },
  { name: "d", age: 22, background: "yellow" },
  { name: "e", age: 23, background: "pink" },
];

const currentIndex = ref(0);
</script>

<template>
  <div>
    <EcSwiper
      :data-list="dataList"
      v-model:currentIndex="currentIndex"
      class="swiper"
    >
      <template v-slot="{ data }">
        <div
          :style="{
            height: '100%',
            background: data.background,
            color: '#fff',
          }"
        >
          <div>{{ data.name }}</div>
          <div>{{ data.age }}</div>
        </div>
      </template>
    </EcSwiper>
    <div>{{ currentIndex }}</div>
  </div>
</template>

<style scoped>
.swiper {
  width: 100%;
  height: 10rem;
}
</style>
