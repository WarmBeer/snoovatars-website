<template>
  <div class="sm:px-2 md:px-6 overflow-x-hidden h-full w-full" ref="container">
    <div class="mx-auto w-full h-full grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 overflow-x-hidden sm:gap-3 divide-y sm:divide-y-0 divide-white/5">
      <template v-for="(item, index) in visibleItems">
        <slot :item="item" :index="index"></slot>
      </template>
    </div>
    <template v-if="visibleItems.length < items.length">
      <div class="py-6 flex flex-col gap-3 items-center">
        <span class="sm:text-lg text-neutral-500">Loading..</span>
        <IonSpinner class="w-5 h-5 text-neutral-500" />
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import {PropType} from "@vue/runtime-core";
import {computed, onBeforeUnmount, onMounted, ref} from "#imports";
import {IonSpinner} from "@ionic/vue";

const props = defineProps({
  items: Array as PropType<any[]>
});

onMounted(() => {
  window.addEventListener('scroll', handleScroll);
});

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll);
});

const loadThreshold = computed(() => {
  return window.innerHeight / 2;
});

const initialBuffer = computed(() => {
  if (window?.innerWidth < 640) {
    return 12;
  }

  return 96;
});

const buffer = computed(() => {
  if (window?.innerWidth < 640) {
    return 24;
  }

  return 72;
});

const container = ref<HTMLInputElement | null>(null);
const endIndex = ref(initialBuffer.value);

function handleScroll() {
  const containerHeight = window?.innerHeight ?? 0;
  const contentHeight = container.value?.scrollHeight ?? 0;
  const position = window?.scrollY ?? 0;

  if (contentHeight - (position + containerHeight) < loadThreshold.value) {
    const newIndex = visibleItems.value.length + buffer.value;

    if (endIndex.value !== newIndex)
      endIndex.value = newIndex;
  } else if (position === 0) {
    endIndex.value = initialBuffer.value;
  }
}

const visibleItems: any[] = computed(() => {
  return props.items.slice(0, endIndex.value);
});
</script>

<style scoped>

</style>
