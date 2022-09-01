<script setup>
import { ref, watchEffect,onMounted } from "vue";

const WIDTH = ref(5);
const HEIGHT = ref(5);
let state = ref([]);
const timer = ref(0);
let Timer;

onMounted(() => {
  startGame();
})

function startGame() {
  if (Timer) clearInterval(Timer);
  timer.value = 0;

  state.value = Array.from({ length: HEIGHT.value }, (_, y) =>
    Array.from({ length: WIDTH.value }, (_, x) => {
      return {
        x: x,
        y: y,
        revealed: false,
        flagged: false,
        mined: false,
        adjacentMines: 0,
      };
    })
  );
  gameStatus.value = "playing";
  mineGenerated = false;
  Timer = setInterval(() => {
    timer.value++;
  }, 1000);
}

let mineGenerated = false;
let dev = ref(false);
const gameStatus = ref("over");

function onClick(block) {
  if (gameStatus.value === "over" || block.flagged) return;
  if (!mineGenerated) {
    generateMines(block);
    mineGenerated = true;
  }
  block.revealed = true;
  if (block.mined) {
    //点到炸弹游戏结束
    alert("Game Over,本次用时" + timer.value + "秒");
    clearInterval(Timer);
    //翻开所有格子，并设置游戏状态为结束
    const blocks = state.value.flat();
    blocks.forEach((block) => {
      block.revealed = true;
    });
    gameStatus.value = "over";
  } else {
    expendZero(block);
  }
}

function onRightClick(block) {
  if (gameStatus.value === "over" || block.revealed) return;
  block.flagged = !block.flagged;
}

// 生成炸弹
function generateMines(initial) {
  for (const row of state.value) {
    for (const block of row) {
      //确保第一次点击的格子四周没有炸弹
      if (Math.abs(initial.x - block.x) < 1) continue;
      if (Math.abs(initial.y - block.y) < 1) continue;
      block.mined = Math.random() < 0.2;
    }
  }
  updateNumbers();
}

//一个格子的3*3范围
const directions = [
  [0, 1],
  [0, -1],
  [1, 0],
  [1, -1],
  [1, 1],
  [-1, 0],
  [-1, 1],
  [-1, -1],
];

//获取当前格子周围的格子
function getSiblings(block) {
  const ret = directions
    .map(([dx, dy]) => {
      const x2 = block.x + dx;
      const y2 = block.y + dy;
      if (x2 < 0 || x2 >= WIDTH.value || y2 < 0 || y2 >= HEIGHT.value) {
        //超出边界
        return undefined;
      }
      return state.value[y2][x2];
    })
    .filter(Boolean);
  return ret;
}

//更新周围炸弹数量
function updateNumbers() {
  state.value.forEach((raw) => {
    raw.forEach((block) => {
      if (block.mined) return;
      getSiblings(block).forEach((b) => {
        if (b.mined) {
          block.adjacentMines++;
        }
      });
    });
  });
}

//展开周围所有没有炸弹的格子
function expendZero(block) {
  if (block.adjacentMines) return;
  getSiblings(block).forEach((s) => {
    if (!s.revealed && !s.flagged) {
      s.revealed = true;
      expendZero(s);
    }
  });
}

const numberColor = [
  "color:#A5D6A7",
  "color:#7CB342",
  "color:#FFF176",
  "color:#FB8C00",
  "color:#F4511E",
  "color:#EC407A",
  "color:#D50000",
];

function getClass(block) {
  if (!block.revealed) return;
  return block.mined
    ? "background-color: #D32F2F;"
    : `background-color:rgba(72, 75, 74, 0.6);${
        numberColor[block.adjacentMines]
      };`;
}

watchEffect(checkGameState);

function checkGameState() {
  if (gameStatus.value === "over") return;
  const blocks = state.value.flat();
  if (blocks.every((block) => block.revealed || block.flagged)) {
    if (blocks.some((block) => block.flagged && !block.mined)) {
      alert("You cheat!");
      clearInterval(Timer);
      gameStatus.value = "over";
    } else {
      alert("You win!,本次用时" + timer.value + "秒");
      clearInterval(Timer);
      gameStatus.value = "over";
    }
  }
}
</script>

<template>
  <div class="wrapper">
    <div class="header-wrapper">
      <span class="title">Minesweeper</span>
      <div class="input-wrapper">
        <input class="num-input" type="text" v-model="WIDTH" />
        <span style="font-size: xx-large">*</span>
        <input class="num-input" type="text" v-model="HEIGHT" />
        <button style="margin-left: 8px" @click="dev = !dev">
          {{ dev ? "Disable cheating mode" : "Enable cheating mode" }}
        </button>
      </div>
      <button @click="startGame" class="start-button">START</button>
    </div>
    <div style="margin-bottom: 10px">Time:{{ timer }}</div>
    <div v-for="(row, y) in state" :key="y" class="col-content">
      <button
        class="mineButton"
        v-for="(item, x) in row"
        :key="x"
        :style="getClass(item)"
        @click="onClick(item)"
        @contextmenu.prevent="onRightClick(item)"
      >
        <template v-if="item.flagged">
          <span>🚩</span>
        </template>
        <template v-else-if="item.revealed || dev">
          <img v-if="item.mined" src="../public/mine.svg" class="mineImg" />
          <div v-else>
            {{ item.adjacentMines === 0 ? "" : item.adjacentMines }}
          </div>
        </template>
      </button>
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  margin: auto;
  align-items: center;
  justify-content: center;
}

.header-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-bottom: 10px;
}

.input-wrapper {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

.num-input {
  display: inline;
  width: 50px;
}

.start-button {
  display: block;
  margin-top: 20px;
}

.mineButton {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0;
  height: 35px;
  width: 35px;
  margin: 1px;
  background-color: rgba(0, 0, 0, 0.2);
  cursor: pointer;
  box-shadow: none;
}
.mineButton:hover {
  background-color: rgba(29, 27, 27, 0.142);
}
.col-content {
  height: 35px;
  display: flex;
}

.title {
  margin-bottom: 20px;
  font-weight: 750;
}
.mineImg {
  height: 20px;
}
</style>
