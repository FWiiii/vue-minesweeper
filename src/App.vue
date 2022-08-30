<script setup>
import { ref, watchEffect } from "vue";

const WIDTH = 10;
const HEIGHT = 10;

const state = ref(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH }, (_, x) => {
      return {
        x: x,
        y: y,
        revealed: false,
        flagged: false,
        mined: false,
        adjacentMines: 0,
      };
    })
  )
);

let mineGenerated = false;
let dev = false;

function onClick(block) {
  if (!mineGenerated) {
    generateMines(block);
    mineGenerated = true;
  }
  block.revealed = true;
  if (block.mined) {
    alert("Game Over");
  }
  expendZero(block);
}

function onRightClick(block) {
  if (block.revealed) {
    return;
  }
  block.flagged = !block.flagged;
}

function generateMines(initial) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) < 1) continue;
      if (Math.abs(initial.y - block.y) < 1) continue;
      block.mined = Math.random() < 0.2;
    }
  }

  updateNumbers();
}

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

function getSiblings(block) {
  return directions
    .map(([dx, dy]) => {
      const x2 = block.x + dx;
      const y2 = block.y + dy;
      if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT) {
        return undefined;
      }
      return state.value[y2][x2];
    })
    .filter(Boolean);
}

function updateNumbers() {
  state.value.forEach((raw, y) => {
    raw.forEach((block, x) => {
      if (block.mined) {
        return;
      }
      getSiblings(block).forEach((b) => {
        if (b.mined) {
          block.adjacentMines++;
        }
      });
    });
  });
}

function expendZero(block) {
  if (block.adjacentMines) {
    return;
  }
  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
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
  if (!block.revealed) {
    return;
  }
  return block.mined
    ? "background-color: #D32F2F;"
    : `background-color:rgba(72, 75, 74, 0.6);${
        numberColor[block.adjacentMines]
      };`;
}

watchEffect(checkGameState)

function checkGameState() {
  const blocks = state.value.flat();
  if (blocks.every(block => block.revealed || block.flagged)){ 
    if (blocks.some(block => block.flagged && !block.mined)) 
      alert("You cheat!");
     else 
      alert("You win!");
    
  }
}
</script>

<template>
  <div class="wrapper">
    <span class="title">Minesweeper</span>
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
.mineButton:hover {
  background-color: rgba(29, 27, 27, 0.142);
}
button {
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
/* button:hover {
  background-color: gray;
} */
.col-content {
  height: 35px;
  display: flex;
}
.wrapper {
  display: flex;
  flex-direction: column;
  margin: 50px auto;
  text-align: center;
}
.title {
  margin-bottom: 10px;
  font-weight: 750;
}
.mineImg {
  height: 20px;
}
</style>
