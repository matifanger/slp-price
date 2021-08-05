<template>
  <v-app>
    <v-main>
      <!-- Leave the container with no paddings and no scrollbar -->
      <v-container fluid class="cont pa-0 ma-0;" >
        <Nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
    }
  }
}
</script>

<style>
::-webkit-scrollbar {display: none;}
</style>