<template>
  <main class="justify-center align-center">
    <table>
      <tr v-for="i in 11">
        <td v-for="i in 13">GRID</td>
      </tr>
    </table>
  </main>
</template>

<script setup>
</script>

<style lang="sass" scoped>
  main
    width: 100vw
    height: 100vh
    background-color: yellow

    table
      border-collapse: collapse
      width: 1300px
      height: 1100px

      td
        text-align: center
        background-color: green
        border: 2px solid blue
</style>