<template>
  <main class="justify-center align-center">
    <section>

      <table>
        <tr v-for="i in ROWS">
          <td v-for="j in COLS" :class="classes[i-1][j-1]">GRID</td>
        </tr>
      </table>

      <div class="player" :style="`top: ${top}px; left: ${left}px`"></div>
    </section>
  </main>
</template>

<script setup>
import { ref } from "vue";
import { COLS, ROWS, SPEED, FPS, MAX_HEIGHT, MAX_WIDTH, FREE_SLOTS, FREE_PCT } from "../config/config";

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

// Order: "WASD"
const flags = [
  [false, () => move(top, -1, MAX_HEIGHT)],
  [false, () => move(left, -1, MAX_WIDTH)],
  [false, () => move(top, 1, MAX_HEIGHT)],
  [false, () => move(left, 1, MAX_WIDTH)]
];

function move(reactive, multiplicator, MAX) {
  const temp = reactive.value + SPEED * multiplicator;

  if (temp < 0) {
    return reactive.value = 0;
  } else if (temp > MAX) {
    return reactive.value = MAX;
  }

  reactive.value = temp;
};

window.addEventListener("keydown", e => {
  const index = "wasd".indexOf(e.key);
  if (index === -1) return;
  flags[index][0] = true;
});

window.addEventListener("keyup", e => {
  const index = "wasd".indexOf(e.key);
  if (index === -1) return;
  flags[index][0] = false;
});

(function loop() {
  flags.forEach(([flag, func]) => {
    if (flag) func();
  });

  setTimeout(loop, UPDATE_TIME);
})();

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
    height: 50px
    width: 50px
    background-color: red

  .free
    background-color: white

  .ice
    background-color: teal

  .ice-unbreakable
    background-color: blue
</style>