<template>
  <div>
    <div class="flex gap-10px">
      <template v-for="(item, index) in betResult" :key="index">
        <div class="border-1 border-solid p-5px rounded-md my-5px">
          {{ item.toFixed(2) }}
        </div>
      </template>
    </div>
    <div class="container center">
      <img
        ref="animatedBox"
        class="w-150px absolute left-0 bottom-0"
        src="@/assets/images/rider_kick.png"
      />
      <div class="flex flex-col items-center z-10">
        <div class="text-40px text-red">{{ num.toFixed(2) }}X</div>
        <div class="text-40px text-green">
          Win Amount: {{ (betAmount * num).toFixed(2) }}
        </div>
      </div>
    </div>
  </div>
  <div class="flex flex-col gap-10px ml-20px w-150px">
    <div>Balance:</div>
    <div>{{ balance.toFixed(2) }}</div>
    <label for="betAmout">BetAmout:</label>
    <input v-model="betAmount" type="number" name="betAmount" id="betAmout" />

    <button
      class="bg-blue rounded-lg border-0 py-10px w-full cursor-pointer"
      :class="{ 'bg-gray cursor-not-allowed': gameStart }"
      :disabled="gameStart"
      @click="playAnimation"
    >
      Play
    </button>
    <button
      class="bg-blue rounded-lg border-0 py-10px w-full cursor-pointer"
      :class="{ 'bg-gray cursor-not-allowed': !gameStart || cashouted }"
      :disabled="!gameStart || cashouted"
      @click="cashout"
    >
      CashOut
    </button>
    <button
      class="bg-blue rounded-lg border-0 py-10px w-full cursor-pointer"
      @click="reset"
    >
      Reset
    </button>
    <div>{{ mutipleNum }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import anime from "animejs";
import Swal from "sweetalert2";

const animatedBox = ref(null);
const animation = ref(null);
const mutipleNum = ref(0);
const num = ref(1.0);
const intervalId = ref(null);
const betAmount = ref(10);
const balance = ref(10000);

const gameStart = ref(false);
const cashouted = ref(false);

const betResult = ref([]);

onMounted(() => {
  animation.value = anime({
    targets: animatedBox.value,
    translateX: 750, // 移動至右側（800px - 50px）
    translateY: -750, // 向上移動至頂部（-800px + 50px）
    rotate: { value: [-45, -45] },
    duration: 50000,
    easing: "easeOutCirc",
    autoplay: false,
  });
});

const playAnimation = () => {
  balance.value -= betAmount.value;
  gameStart.value = true;
  cashouted.value = false;
  animation.value.play();
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
  num.value = 1.0;
  animation.value.restart();
  animation.value.pause();
  num.value = 1.0;
  clearInterval(intervalId.value);
  intervalId.value = null;
};

const cashout = () => {
  cashouted.value = true;
  balance.value += betAmount.value * mutipleNum.value;
  Swal.fire({
    title: "You Won!!!",
    text: betAmount.value * mutipleNum.value,
    icon: "success",
  });
};
watch(
  () => num.value,
  (newVal) => {
    if (newVal >= mutipleNum.value) {
      clearInterval(intervalId.value);
      intervalId.value = null;
      betResult.value.push(mutipleNum.value);
      Swal.fire({
        title: "Henshin!!!",
        icon: "warning",
      });
      reset();
    }
  }
);
</script>

<style>
.container {
  width: 800px;
  height: 800px;
  position: relative;
  background-color: #f0f0f0;
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
</style>
