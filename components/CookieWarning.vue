<template>
  <div class="px-2 md:px-4 py-2 fixed bottom-14 w-full text-header justify-center items-center max-w-lg z-50">
    <div class="p-6 flex justify-between bg-neutral-700 rounded-lg shadow-2xl">
      <span>RCAX uses cookies to analyze traffic.</span>
      <button @click="acceptCookies" class="text-amber-500 hover:text-amber-400">OK</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import {useSettings} from "~/composables/states";

const settings = useSettings();

function acceptCookies() {
  settings.value.cookies = {
    accepted: true
  };
}
</script>

<style>

</style>
