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

const classes = [...Array(ROWS).keys()].map(i => [
  ...[...Array(COLS).keys()].map(j => {
    if (FREE_SLOTS.includes(`${i}-${j}`)) return "free";
    if (i % 2 && j % 2) return "ice-unbreakable";
    if (Math.random() < FREE_PCT) return "ice";
    return "free";
  })
]);

const top = ref(0);
const left = ref(0);
const power = ref(1);

let bombs_available = 1;
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
      if (classes[X][Y] !== "free") return;
    }
  };

  reactive.value = temp;
};

function placeBomb() {
  if (bombs.length === bombs_available) return;
  const [cellX, cellY] = getCell(top.value, left.value);
  bombs.push([cellX, cellY]);
  setTimeout(animateBomb, BOMB_TIME - 1000);
  setTimeout(clearBomb, BOMB_TIME);
};

function animateBomb() {
  const bomb = document.getElementsByClassName("bomb")[0];
  bomb.classList.add("bomb-explosion");
};

function clearBomb() {
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
      animation: explosion-top 1s linear 2s forwards
      top: calc(-100px * var(--power))
      height: calc(200px * var(--power) + 50px)
      width: 50px

    &::after
      left: calc(-100px * var(--power))
      height: 50px
      width: calc(200px * var(--power) + 50px)

  .free
    background-color: white

  .ice
    background-color: teal

  .ice-unbreakable
    background-color: blue
</style>