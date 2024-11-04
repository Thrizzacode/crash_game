<template>
  <div
    class="flex flex-col gap-20px w-150px text-#5f6892 bg-#200731 p-20px w-400px rounded-lg mr-20px text-18px font-bold"
  >
    <div>Balance:</div>
    <div class="text-white">{{ balance.toFixed(2) }}</div>
    <label for="betAmout">BetAmout:</label>
    <div class="relative">
      <input
        v-model="betAmount"
        type="number"
        name="betAmount"
        id="betAmout"
        :disabled="isBet || isGameStarted"
        class="bg-#200790/50 text-white h-40px w-full rounded-lg border-0 p-10px text-18px"
      />
      <div class="absolute top-1/2 right-10px -translate-y-1/2 flex gap-5px">
        <button
          class="bg-#200745 py-8px px-10px rounded-md text-#5f6892 border-0 cursor-pointer hover:text-white"
          @click="half"
        >
          1/2
        </button>
        <button
          class="bg-#200745 py-8px px-10px rounded-md text-#5f6892 border-0 cursor-pointer hover:text-white"
          @click="double"
        >
          x2
        </button>
        <button
          class="bg-#200745 py-8px px-10px rounded-md text-#5f6892 border-0 cursor-pointer hover:text-white"
          @click="max"
        >
          Max
        </button>
      </div>
    </div>

    <button
      class="btn rounded-lg border-0 py-10px w-full cursor-pointer"
      :class="{ 'bg-gray cursor-not-allowed': isBet || isGameStarted }"
      :disabled="isBet || isGameStarted"
      @click="betAnimation"
    >
      Bet
    </button>
    <button
      class="btn rounded-lg border-0 py-10px w-full cursor-pointer"
      :class="{ 'bg-gray cursor-not-allowed': !isBet || cashouted || !isGameStarted }"
      :disabled="!isBet || cashouted || !isGameStarted"
      @click="cashout"
    >
      CashOut
    </button>
    <!-- <button class="btn rounded-lg border-0 py-10px w-full cursor-pointer" @click="reset">
      Reset
    </button> -->
    <div>{{ result }}</div>
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
        <img v-if="isCrashed" class="w-200px" src="@/assets/images/bomb.png" />
        <img v-else class="w-150px" src="@/assets/images/rocket.png" />
      </div>
      <div class="flex flex-col items-center z-10 mt-15%">
        <div class="text-80px text-red font-bold">{{ multipleNum.toFixed(2) }}X</div>
        <div class="text-40px text-green font-bold">
          Current Cashout: {{ (betAmount * multipleNum).toFixed(2) }}
        </div>
        <div>
          <div v-if="countDown > 0" class="text-40px text-white font-bold">
            Countdown: {{ countDown }}
          </div>
        </div>
        <div v-if="isCrashed" class="text-100px text-red font-bold">Crash!!!</div>
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
const multipleNum = ref(0);
const betAmount = ref(10);
const balance = ref(10000);

const isGameStarted = ref(false);
const isBet = ref(false);
const cashouted = ref(false);
const isCrashed = ref(false);

const betResult = ref([]);

// WebSocket
// onMounted(() => {
// const socket = new WebSocket('ws://localhost:3000');

// socket.onopen = () => {
//   console.log('Connected to server');
//   // 讓玩家加入遊戲
//   socket.send(JSON.stringify({ type: 'join' }));
// };

// socket.onmessage = (event) => {
//   const data = JSON.parse(event.data);
//   switch (data.type) {
//     case 'join':
//       console.log('Join game');
//       break;
//     case 'result':
//       console.log(`Game result: ${data.result}`);
//       break;
//     case 'countdownStart':
//       animation.value.restart();
//       animation.value.pause();
//       curveAnimation.value.restart();
//       curveAnimation.value.pause();
//       isCrashed.value = false;
//       isBet.value = false;
//       break;
//     case 'countdown':
//       countDown.value = data.countdown;
//       console.log(`Game countdown: ${data.countdown}`);
//       break;
//     case 'gameStart':
//       console.log('Game start');
//       animation.value.play();
//       curveAnimation.value.play();
//       isBet.value = true;
//       result.value = data.result;
//       break;
//     case 'multiplier':
//       console.log(parseFloat(data.multiplier));
//       multipleNum.value = parseFloat(data.multiplier);
//       break;
//     case 'crash':
//       console.log('Game crash');
//       animation.value.pause();
//       curveAnimation.value.pause();
//       betResult.value.push(multipleNum.value);
//       isCrashed.value = true;
//       break;
//     default:
//       break;
//   }
// };

// socket.onclose = () => {
//   console.log('Disconnected from server');
// };
// })

// 开始倒计时
const isCountingDown = ref(false);
const countDown = ref(0);
let countdownInterval = null;

const startCountdown = () => {
  isCountingDown.value = true;
  countDown.value = 5;
  animation.value.restart();
  animation.value.pause();
  curveAnimation.value.restart();
  curveAnimation.value.pause();
  isGameStarted.value = false;
  isCrashed.value = false;
  isBet.value = false;
  countdownInterval = setInterval(() => {
    console.log('Countdown', countDown.value);
    countDown.value--;
    if (countDown.value < 0) {
      clearInterval(countdownInterval);
      isCountingDown.value = false;
      startGame();
    }
  }, 1000);
};

// 开始游戏
const result = ref(0);
let gameInterval = null;

const startGame = () => {
  // 生成随机结果
  result.value = 0.99 / (1 - Math.random());
  console.log('Game started, result is', result.value);
  let currentMultiplier = 1.0;
  isGameStarted.value = true;
  animation.value.play();
  curveAnimation.value.play();
  // 累加倍率
  gameInterval = setInterval(() => {
    currentMultiplier += 0.01;
    multipleNum.value = currentMultiplier;
    if (currentMultiplier >= result.value) {
      // 触发崩溃
      console.log('Crash!');
      animation.value.pause();
      curveAnimation.value.pause();
      betResult.value.push(multipleNum.value);
      isCrashed.value = true;
      clearInterval(gameInterval);
      // 五秒后重新开始倒计时
      setTimeout(() => {
        startCountdown();
      }, 5000);
    }
  }, 100); // 每100毫秒累加一次
};

const betAnimation = () => {
  balance.value -= betAmount.value;
  isBet.value = true;
  cashouted.value = false;
};

const cashout = () => {
  cashouted.value = true;
  balance.value += betAmount.value * multipleNum.value;
  Swal.fire({
    title: 'You Won!!!',
    text: (betAmount.value * multipleNum.value).toFixed(2),
    icon: 'success',
  });
};

// 投注算金額計
// const checkBalance = () => {
//   if (betAmount.value > balance.value) {
//     console.log('eee');
//     Swal.fire({
//       title: 'Insufficient balance',
//       icon: 'error',
//     });
//     return true;
//   } else {
//     return false;
//   }
// };

/**
 * 將投注金額減半
 * 會先檢查餘額，並確保不低於最小投注限制
 */
const half = () => {
  // 檢查餘額是否足夠
  if (isGameStarted.value) return;

  // 計算新的投注金額
  const minBet = 1; // 最小投注金額
  const newAmount = Math.floor(betAmount.value / 2);

  // 確保不低於最小投注金額
  betAmount.value = Math.max(newAmount, minBet);
};

/**
 * 將投注金額倍增
 * 會先檢查餘額，並確保不超過餘額和最大投注限制
 */
const double = () => {
  if (isGameStarted.value) return;

  const maxBet = balance.value; // 最大投注金額為當前餘額
  const doubledAmount = Math.floor(betAmount.value * 2);

  betAmount.value = Math.min(doubledAmount, maxBet);
};

/**
 * 將投注金額設為最大可投注金額
 */
const max = () => {
  if (isGameStarted.value) return;
  betAmount.value = Math.floor(balance.value);
};

onMounted(() => {
  const path = anime.path('.container #rocketPath path');
  animation.value = anime({
    targets: animatedBox.value,
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

  // 初始时启动倒计时
  startCountdown();
});
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
