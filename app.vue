<template>
  <NuxtPage />
</template>

<script setup>
const onResize = () => {
  const vh = window.innerHeight * 0.01;
  document.documentElement.style.setProperty("--vh", `${vh}px`);
};

onBeforeMount(() => {
  console.clear();
});

onMounted(() => {
  onResize();
  window.addEventListener("resize", onResize);
});
</script>
