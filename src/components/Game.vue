<template>
  <main class="justify-center align-center">
    <section>

      <table>
        <tr v-for="i in 11">
          <td v-for="i in 13">GRID</td>
        </tr>
      </table>

      <div class="player" :style="`top: ${top}px; left: ${left}px`"></div>
    </section>
  </main>
</template>

<script setup>
import { ref } from "vue";
import { SPEED, FPS, MAX_HEIGHT, MAX_WIDTH } from "../config/config";

const top = ref(0);
const left = ref(0);

// Order: "WASD"
const flags = [
  [false, () => add(top, -1, MAX_HEIGHT)],
  [false, () => add(left, -1, MAX_WIDTH)],
  [false, () => add(top, 1, MAX_HEIGHT)],
  [false, () => add(left, 1, MAX_WIDTH)]
];

function add(reactive, multiplicator, MAX) {
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

  setTimeout(loop, 1 / FPS);
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
          background-color: green
          border: 2px solid blue

  .player
    position: absolute
    height: 50px
    width: 50px
    background-color: red
</style>