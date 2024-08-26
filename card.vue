<script setup lang="ts">
import { ref, computed } from "vue";

const props = defineProps<{
  dataList: {
    color: string;
  }[];
}>();

const emit = defineEmits<{
  (e: "slideCard", cw: number, ex: number, ey?: number): void;
  (e: "moveCard", cw: number, dx: number, dy?: number): void;
  (e: "moveEndCard", cw: number, dx: number, dy?: number): void;
}>();

const base = [{ color: "black" }];
const cardStack = computed(() => base.concat(props.dataList).slice(-4));

const cardSetRef = ref<HTMLElement | null>(null);

// through `eventLock` to prevent multiple touch events
let eventLock = false;
const ontouchstart = (e: TouchEvent) => {
  if (eventLock) return;
  const target = e.target as HTMLElement;
  const cw = target.clientWidth / 2.8;
  const sx = e.touches[0].clientX,
    sy = e.touches[0].clientY;
  let dx = 0,
    dy = 0,
    deg = 0;

  target.classList.add("move");

  const ontouchmove = (e: TouchEvent) => {
    dx = e.touches[0].clientX - sx;
    dy = e.touches[0].clientY - sy;

    emit("moveCard", cw, dx, dy);
    deg = dx > 0 ? (dy / sy) * 3 : (dy / sy) * -6;
    target.style.transform = `translate3D(${dx}px, ${dy}px, 0) rotate(${deg}deg)`;
  };
  const ontouchend = () => {
    if (dx !== 0 || dy !== 0) {
      eventLock = true;
      target.classList.add("active");
      if (dx > cw || dx < -cw) {
        target.style.transform = `translate3D(${dx > 0 ? 100 : -100}vw, ${dy}px, 0) rotate(${deg}deg)`;
      } else {
        target.style.transform = "none";
      }
    } else {
      target.classList.remove("move");
    }
    emit("moveEndCard", cw, dx, dy);
    target.addEventListener("transitionend", ontransitionend);
    document.removeEventListener("touchmove", ontouchmove);
    document.removeEventListener("touchend", ontouchend);
  };
  const ontransitionend = () => {
    eventLock = false;
    emit("slideCard", cw, dx, dy);

    target.classList.remove("active");
    target.classList.remove("move");
    target.removeEventListener("transitionend", ontransitionend);
  };

  document.addEventListener("touchmove", ontouchmove);
  document.addEventListener("touchend", ontouchend);
};
</script>

<template>
  <div
    ref="cardSetRef"
    class="card-set"
  >
    <div
      v-for="card in cardStack"
      :key="card.color"
      class="card"
      :style="{ backgroundColor: card.color }"
      @touchstart.prevent="ontouchstart"
    ></div>
  </div>
</template>

<style scoped lang="less">
.card-set {
  width: 100%;
  height: 590px;
  position: relative;
  .card {
    z-index: 2;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    margin: 0 auto;
    width: 343px;
    height: 545px;
    border-radius: 20px;
    overflow: hidden;
    transition: 0.3s;
    transform-origin: top center;
    pointer-events: none;

    &.move {
      transition: none;
    }

    &.active {
      transition: 0.3s;
    }

    &:nth-last-of-type(1) {
      top: 28px;
      pointer-events: auto;
    }
    &:nth-last-of-type(2) {
      top: 20px;
      transform: scale(0.93);
    }
    &:nth-last-of-type(3),
    &:nth-last-of-type(4) {
      top: 12px;
      transform: scale(0.854);
    }
    &:nth-last-of-type(4) {
      transform: scale(0.84);
    }
  }
}
</style>
