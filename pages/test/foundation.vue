<template>
  <div class="container">
    <div ref="canvasRef" class="canvas" />
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

const canvasRef = ref();

let renderer;
let scene;
let camera;
let raf;

let controls;
function init() {
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
  canvasRef.value.appendChild(renderer.domElement);

  controls = new OrbitControls(camera, renderer.domElement);
}

function animate() {
  camera.updateMatrixWorld();
  renderer.render(scene, camera);
  controls.update();

  raf = requestAnimationFrame(animate);
}

const onResize = () => {
  camera.aspect = canvasRef.value.offsetWidth / canvasRef.value.offsetHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
};

onMounted(() => {
  scene = new THREE.Scene();
  renderer = new THREE.WebGLRenderer({ antialias: true });

  camera = new THREE.PerspectiveCamera(
    70,
    canvasRef.value.offsetWidth / canvasRef.value.offsetHeight,
    0.1,
    10,
  );

  init();
  animate();

  window.addEventListener("resize", onResize);
});

onBeforeUnmount(() => {
  cancelAnimationFrame(raf);
  renderer.dispose();

  window.removeEventListener("resize", onResize);
});
</script>

<style lang="scss" scoped>
.container {
  width: 100%;
  height: 100vh;
  .canvas {
    position: absolute;
    width: 100%;
    height: 100%;
  }
}
</style>
