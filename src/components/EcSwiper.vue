<script lang="ts" setup>
import { ref } from "vue";
import useSwiper from "../hooks/useSwiper";
import { computed } from "@vue/reactivity";

const props = defineProps<{
  dataList: any[];
  currentIndex: number;
  calcPreviousPosition?: (currentPosition: number) => number;
  calcNextPosition?: (currentPosition: number) => number;
}>();
const emit = defineEmits(["update:currentIndex"]);

const swiperCurrentIndex = computed({
  get() {
    return props.currentIndex;
  },
  set(value) {
    emit("update:currentIndex", value);
  },
});

const ecSwiper = ref<HTMLDivElement | null>(null);
const {
  currentIndex,
  prevIndex,
  nextIndex,
  currentPosition,
  prevPosition,
  nextPosition,
} = useSwiper({
  containerRef: ecSwiper,
  dataLength: props.dataList.length,
  currentIndex: swiperCurrentIndex,
  onIndexChanged: (index, oldIndex) => {
    console.log(`${oldIndex} -> ${index}`);
  },
});
</script>

<template>
  <div ref="ecSwiper" class="ec-swiper">
    <div class="ec-swiper__item ec-swiper__item--prev">
      <slot :data="props.dataList[prevIndex]"></slot>
    </div>
    <div class="ec-swiper__item ec-swiper__item--cur">
      <slot :data="props.dataList[currentIndex]"></slot>
    </div>
    <div class="ec-swiper__item ec-swiper__item--next">
      <slot :data="props.dataList[nextIndex]"></slot>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.ec-swiper {
  position: relative;
  width: 100%;
  height: 10rem;
  overflow: hidden;

  &__item {
    position: absolute;
    width: 100%;
    height: 100%;

    &--prev {
      transform: translateX(v-bind("prevPosition+'px'"));
    }
    &--cur {
      transform: translateX(v-bind("currentPosition+'px'"));
    }
    &--next {
      transform: translateX(v-bind("nextPosition+'px'"));
    }
  }
}
</style>
