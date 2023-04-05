<script setup lang="ts">
import { computed, onMounted, reactive, ref } from "vue";
import useGesture from "../hooks/useGesture";

const props = defineProps<{
  dataList: any[];
  currentIndex: number;
  calcPreviousPosition?: (currentPosition: number) => number;
  calcNextPosition?: (currentPosition: number) => number;
}>();
const emit = defineEmits(["update:currentIndex"]);

// 页面数据下标
const currentIndex = computed({
  get() {
    return props.currentIndex;
  },
  set(value) {
    emit("update:currentIndex", value);
  },
});
const previousIndex = computed(() => {
  return currentIndex.value - 1 < 0
    ? props.dataList.length - 1
    : currentIndex.value - 1;
});
const nextIndex = computed(() => {
  return currentIndex.value + 1 > props.dataList.length - 1
    ? 0
    : currentIndex.value + 1;
});

const swiperRef = ref<HTMLElement | null>(null);

// 容器尺寸
const swiperSize = reactive({ width: 0 });

const dragData = reactive({
  dragging: false,
  startX: 0,
  moveX: 0,
  endX: 0,
});

/**
 * 计算上一个滑动位置
 * @param currentPosition 当前滑动块位置
 * @returns 上一个滑动块位置
 */
const calcPreviousPosition =
  props.calcPreviousPosition ||
  ((currentPosition: number) => currentPosition - swiperSize.width);

/**
 * 计算下一个滑动位置
 * @param currentPosition 当前滑动块位置
 * @returns 下一个滑动块位置
 */
const calcNextPosition =
  props.calcNextPosition ||
  ((currentPosition: number) => currentPosition + swiperSize.width);

const currentAnimationPosition = ref(0);
const previousAnimationPosition = computed(() =>
  calcPreviousPosition(currentAnimationPosition.value)
);
const nextAnimationPosition = computed(() =>
  calcNextPosition(currentAnimationPosition.value)
);

// 内部元素位置
const swiperItemsPosition = computed(() => {
  if (dragData.dragging) {
    const distance = dragData.moveX - dragData.startX;

    const current = distance;
    const previous = calcPreviousPosition(current);
    const next = calcNextPosition(current);

    return {
      previous,
      current,
      next,
    };
  } else {
    return {
      previous: previousAnimationPosition.value,
      current: currentAnimationPosition.value,
      next: nextAnimationPosition.value,
    };
  }
});

// 初始化容器尺寸
onMounted(() => {
  swiperSize.width = swiperRef.value!.offsetWidth;
});

// 根据滑动方向和距离计算当前索引
function calculateCurrentIndexAndSwitch() {
  const distance = dragData.endX - dragData.startX;

  // 获取滑动方向 -1: 向左滑动，下一页 1: 向右滑动，上一页
  const direction = distance > 0 ? -1 : 1;
  currentIndex.value = direction === -1 ? previousIndex.value : nextIndex.value;

  // 计算动画位置
  const offset = direction * swiperSize.width + distance;
  currentAnimationPosition.value = swiperItemsPosition.value.current + offset;

  animationRestorePosition();
}

// 动画来恢复位置
function animationRestorePosition() {
  requestAnimationFrame(() => {
    if (dragData.dragging) return;

    // 滑动距离小于5px时，停止动画
    const distance = currentAnimationPosition.value;
    if (Math.abs(distance) < 5) {
      currentAnimationPosition.value = 0;
      return;
    }

    // 每次滑动距离减少1/10
    const offset = distance / 1.1;

    currentAnimationPosition.value = offset;
    animationRestorePosition();
  });
}

// 手势
const { onStart, onMove, onEnd } = useGesture(swiperRef);

onStart((x) => {
  dragData.dragging = true;
  dragData.startX = x - currentAnimationPosition.value;
  dragData.moveX = x;
});

onMove((x) => {
  dragData.moveX = x;
});

onEnd((x) => {
  dragData.dragging = false;
  dragData.endX = x;

  calculateCurrentIndexAndSwitch();

  // 重置
  dragData.startX = 0;
  dragData.moveX = 0;
  dragData.endX = 0;
});
</script>

<template>
  <div ref="swiperRef" class="swiper">
    <div class="swiper-item swiper-item--previous">
      <slot :data="props.dataList[previousIndex]"></slot>
    </div>
    <div class="swiper-item swiper-item--current">
      <slot :data="props.dataList[currentIndex]"></slot>
    </div>
    <div class="swiper-item swiper-item--next">
      <slot :data="props.dataList[nextIndex]"></slot>
    </div>
  </div>
</template>

<style scoped>
.swiper {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.swiper-item {
  width: 100%;
  height: 100%;
}

.swiper-item--previous {
  position: absolute;
  left: 0;
  top: 0;
  transform: translateX(v-bind("swiperItemsPosition.previous+'px'"));
  /* background-color: blue; */
}

.swiper-item--current {
  position: absolute;
  left: 0;
  top: 0;
  transform: translateX(v-bind("swiperItemsPosition.current+'px'"));
  /* background-color: rebeccapurple; */
}
.swiper-item--next {
  position: absolute;
  left: 0;
  top: 0;
  transform: translateX(v-bind("swiperItemsPosition.next+'px'"));
  /* background-color: yellow; */
}
</style>
