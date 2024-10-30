<template>
  <div
    class="flex flex-col gap-20px w-150px text-#5f6892 bg-#200731 p-20px w-400px rounded-lg mr-20px text-18px font-bold"
  >
    <div>Balance:</div>
    <div class="text-white">{{ balance.toFixed(2) }}</div>
    <label for="betAmout">BetAmout:</label>
    <input
      v-model="betAmount"
      type="number"
      name="betAmount"
      id="betAmout"
      class="bg-#200790/50 text-white h-30px w-full rounded-lg border-0 p-5px text-18px"
    />

    <button
      class="btn rounded-lg border-0 py-10px w-full cursor-pointer"
      :class="{ 'bg-gray cursor-not-allowed': gameStart }"
      :disabled="gameStart"
      @click="playAnimation"
    >
      Play
    </button>
    <button
      class="btn rounded-lg border-0 py-10px w-full cursor-pointer"
      :class="{ 'bg-gray cursor-not-allowed': !gameStart || cashouted }"
      :disabled="!gameStart || cashouted"
      @click="cashout"
    >
      CashOut
    </button>
    <button class="btn rounded-lg border-0 py-10px w-full cursor-pointer" @click="reset">
      Reset
    </button>
    <div>{{ mutipleNum }}</div>
  </div>

  <div>
    <div class="flex gap-10px h-46px">
      <template v-for="(item, index) in betResult" :key="index">
        <div class="border-1 border-solid p-5px rounded-md my-5px text-white">
          {{ item.toFixed(2) }}
        </div>
      </template>
    </div>
    <div class="container flex justify-center">
      <svg
        id="curve"
        xmlns="http://www.w3.org/2000/svg"
        class="absolute left-0 bottom-0 w-600px h-600px"
        viewBox="0 0 400 400"
      >
        <path
          d="M 0 400 Q 400 400 400 0"
          stroke="#fdaa10"
          stroke-width="4"
          fill="none"
          class="glow"
        />
      </svg>
      <svg
        id="rocketPath"
        xmlns="http://www.w3.org/2000/svg"
        class="absolute left-0 bottom-0 w-600px h-600px"
        viewBox="0 0 400 400"
      >
        <path d="M 0 0 Q 400 0 400 -400" stroke="#000" stroke-width="0" fill="none" />
      </svg>

      <div ref="animatedBox" class="absolute -left-75px -bottom-75px">
        <img v-if="crashed" class="w-200px" src="@/assets/images/bomb.png" />
        <img v-else class="w-150px" src="@/assets/images/rocket.png" />
      </div>
      <div class="flex flex-col items-center z-10 mt-15%">
        <div class="text-80px text-red font-bold">{{ num.toFixed(2) }}X</div>
        <div class="text-40px text-green font-bold">
          Current Cashout: {{ (betAmount * num).toFixed(2) }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import anime from 'animejs';
import Swal from 'sweetalert2';

const animatedBox = ref(null);
const animation = ref(null);
const curveAnimation = ref(null);
const mutipleNum = ref(0);
const num = ref(1.0);
const intervalId = ref(null);
const betAmount = ref(10);
const balance = ref(10000);

const gameStart = ref(false);
const cashouted = ref(false);
const crashed = ref(false);

const betResult = ref([]);

onMounted(() => {
  console.log(animatedBox.value);
  const path = anime.path('.container #rocketPath path');
  animation.value = anime({
    targets: animatedBox.value,
    // translateX: 650, // 移動至右側（800px - 50px）
    // translateY: -650, // 向上移動至頂部（-800px + 50px）
    // rotate: { value: [-45, -45] },
    translateX: path('x'),
    translateY: path('y'),
    rotate: { value: [0, -20] },
    duration: 5000,
    easing: 'linear',
    autoplay: false,
  });

  curveAnimation.value = anime({
    targets: '.container #curve path',
    strokeDashoffset: [anime.setDashoffset, 0],
    duration: 5000,
    easing: 'linear',
    autoplay: false,
  });
});

const playAnimation = () => {
  balance.value -= betAmount.value;
  gameStart.value = true;
  cashouted.value = false;
  animation.value.play();
  curveAnimation.value.play();
  mutipleNumGenerator();
  startIncrement();
};

const restartAnimation = () => {
  animation.value.restart();
  mutipleNumGenerator();
  num.value = 1.0;
  clearInterval(intervalId.value);
  intervalId.value = null;
  startIncrement();
};

const mutipleNumGenerator = () => {
  mutipleNum.value = 0.99 / (1 - Math.random());
};

const startIncrement = () => {
  if (intervalId.value) return; // 防止多次点击启动多个计时器
  intervalId.value = setInterval(() => {
    num.value += 0.01;
  }, 100); // 每100毫秒增加0.01
};

const reset = () => {
  gameStart.value = false;
  crashed.value = false;
  num.value = 1.0;
  animation.value.restart();
  animation.value.pause();
  curveAnimation.value.restart();
  curveAnimation.value.pause();
  num.value = 1.0;
  clearInterval(intervalId.value);
  intervalId.value = null;
};

const cashout = () => {
  cashouted.value = true;
  balance.value += betAmount.value * num.value;
  Swal.fire({
    title: 'You Won!!!',
    text: (betAmount.value * num.value).toFixed(2),
    icon: 'success',
  });
};
watch(
  () => num.value,
  (newVal) => {
    if (newVal >= mutipleNum.value) {
      clearInterval(intervalId.value);
      intervalId.value = null;
      betResult.value.push(mutipleNum.value);
      crashed.value = true;
      cashouted.value = true;
      animation.value.pause();
      curveAnimation.value.pause();
      Swal.fire({
        title: 'Crashed!!!',
        icon: 'warning',
      });
    }
  }
);
</script>

<style>
.container {
  position: relative;
  width: 800px;
  height: 800px;
  position: relative;
  background: rgb(64, 17, 75);
  background: linear-gradient(
    174deg,
    rgba(64, 17, 75, 1) 0%,
    rgba(121, 9, 34, 1) 65%,
    rgba(122, 19, 138, 1) 100%
  );
  border-radius: 10px;
  overflow: hidden;
}

.animated-box {
  width: 20px;
  height: 20px;
  position: absolute;
  left: 0;
  bottom: 0;
  background-color: #3498db;
}

.btn {
  font-size: 18px;
  background-color: #fdaa10;
  transition: all 0.18s ease-in-out;
}

.btn:hover {
  text-shadow: 0 0 30px rgba(255, 255, 255, 1), 0 0 60px rgba(255, 255, 255, 0.8),
    0 0 75px rgba(255, 255, 255, 0.6), 0 0 80px rgba(255, 255, 255, 0.4),
    0 0 77px rgba(255, 255, 255, 0.5), 0 0 78px rgba(255, 255, 255, 0.4),
    0 0 79px rgba(255, 255, 255, 0.3), 0 0 80px rgba(255, 255, 255, 0.2),
    0 0 120px rgba(255, 255, 255, 0.1);
}

/* #curve path {
  stroke-dasharray: 50;
} */
.glow {
  filter: drop-shadow(0 0 5px blue) drop-shadow(0 0 10px blue);
}
</style>
