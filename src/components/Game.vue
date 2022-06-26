<template>
  <main class="justify-center align-center">
    <section>
      <table>
        <tr v-for="i in ROWS">
          <td v-for="j in COLS" :class="classes[i-1][j-1]"></td>
        </tr>
      </table>

      <div class="player" :style="`top: ${top}px; left: ${left}px`"></div>
      <div>
        <div v-for="[x, y] in bombs" :key="`${x}-${y}`" class="bomb" :style="`top: ${x*100+25}px; left: ${y*100+25}px; --power: ${power}`"></div>
      </div>
    </section>
  </main>
</template>

<script setup>
import { reactive, ref } from "vue";
import { COLS, ROWS, SPEED, FPS, MAX_HEIGHT, MAX_WIDTH, FREE_SLOTS, BOMB_TIME, FREE_PCT } from "../config/config";

const UPDATE_TIME = 1000 / FPS;

const classes = reactive([...Array(ROWS).keys()].map(i => [
  ...[...Array(COLS).keys()].map(j => {
    if (FREE_SLOTS.includes(`${i}-${j}`)) return "free";
    if (i % 2 && j % 2) return "ice-unbreakable";
    if (Math.random() < FREE_PCT) return "ice";
    return "free";
  })
]));

const top = ref(0);
const left = ref(0);
const power = ref(1);

let bombs_available = 1;
let dead = false;
let toClear = [];
const bombs = reactive([]);

// Order: "WASD"
const flags = [
  [false, () => move("top", -1)],
  [false, () => move("left", -1)],
  [false, () => move("top", 1)],
  [false, () => move("left", 1)]
];

// Get all cells the player is currently in (4 total)
function getCells(x, y) {
  return [
    [
      Math.floor(x / 100),
      Math.ceil(x / 100)
    ],

    [
      Math.floor(y / 100),
      Math.ceil(y / 100)
    ]
  ];
};

// Get the closest cell
function getCell(x, y) {
  return [Math.round(x / 100), Math.round(y / 100)];
}

function move(direction, multiplicator) {
  if (dead) return;
  const reactive = direction === "top" ? top : left;
  const MAX = direction === "top" ? MAX_HEIGHT : MAX_WIDTH;

  const temp = reactive.value + SPEED * multiplicator;

  if (temp < 0) {
    return reactive.value = 0;
  } else if (temp > MAX) {
    return reactive.value = MAX;
  };

  const x = direction === "top" ? temp : top.value;
  const y = direction === "left" ? temp : left.value;

  const [cellX, cellY] = getCells(x, y);

  for (const X of cellX) {
    for (const Y of cellY) {
      if (classes[X][Y] === "free-dead") return dead = true;
      if (classes[X][Y] !== "free") return;
    }
  };

  reactive.value = temp;
};

function placeBomb() {
  if (bombs.length === bombs_available) return;
  const [cellX, cellY] = getCell(top.value, left.value);
  bombs.push([cellX, cellY]);
  setTimeout(animateBomb, BOMB_TIME);
  setTimeout(clearBomb, BOMB_TIME + 1000);
};

function animateBomb() {
  const [bombX, bombY] = bombs[0];
  const [cellX, cellY] = getCells(top.value, left.value);
  toClear.push([]);
  const index = toClear.length - 1;

  for (let bombPower = -power.value; bombPower <= power.value; bombPower++) {
    const blockX = bombX + bombPower;
    if (blockX < 0) continue;
    if (classes[blockX][bombY] !== "ice-unbreakable") {
      classes[blockX][bombY] = "free-dead";
      toClear[index].push([blockX, bombY]);
    };
    if (cellX.includes(blockX) && cellY.includes(bombY)) dead = true;
  };

  for (let bombPower = -power.value; bombPower <= power.value; bombPower++) {
    const blockY = bombY + bombPower;
    if (blockY < 0) continue;
    if (classes[bombX][blockY] !== "ice-unbreakable") {
      classes[bombX][blockY] = "free-dead";
      toClear[index].push([bombX, blockY]);
    }
    if (cellX.includes(bombX) && cellY.includes(blockY)) dead = true;
  };

  document.getElementsByClassName("bomb")[0].classList.add("bomb-explosion");
};

function clearBomb() {
  for (const [x, y] of toClear[0]) classes[x][y] = "free";
  toClear.shift();
  bombs.shift();
};

window.addEventListener("keydown", e => {
  if (e.key === " ") return placeBomb();
  const index = "wasd".indexOf(e.key);
  if (index === -1) return;
  flags[index][0] = true;
});

window.addEventListener("keyup", e => {
  const index = "wasd".indexOf(e.key);
  if (index === -1) return;
  flags[index][0] = false;
});

function loop() {
  flags.forEach(([flag, func]) => {
    if (flag) func();
  });
}

setInterval(loop, UPDATE_TIME);

</script>

<style lang="sass" scoped>
  main
    width: 100vw
    height: 100vh
    background-color: yellow

    section
      position: relative
      width: 1300px
      height: 1100px

      table
        width: 100%
        height: 100%
        border-collapse: collapse

        td
          text-align: center
          border: 2px solid blue

  .player
    position: absolute
    height: 100px
    width: 100px
    background-color: red

  .bomb
    position: absolute
    height: 50px
    width: 50px
    background-color: yellow

  .bomb-explosion
    &::before, &::after
      content: ""
      position: absolute
      border-radius: 50px
      background-color: orange

    &::before
      left: 0
      top: calc(-100px * var(--power))
      height: calc(200px * var(--power) + 50px)
      width: 50px

    &::after
      left: calc(-100px * var(--power))
      height: 50px
      width: calc(200px * var(--power) + 50px)

  .free, .free-dead
    background-color: white

  .ice
    background-color: teal

  .ice-unbreakable
    background-color: blue
</style>