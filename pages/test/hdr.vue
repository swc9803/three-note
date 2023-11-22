<template>
  <div class="wrapper">
    <div ref="containerRef" class="container" />
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";

const containerRef = ref();
let camera;
let raf;

const scene = new THREE.Scene();
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1;

// model
const geometry = new THREE.TorusKnotGeometry(1, 0.3, 256, 64, 2, 3);
const material = new THREE.MeshStandardMaterial({ color: 0xffffff });
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// background + hdr light
let hdr;
new RGBELoader().load("/brown_photostudio_03_4k.hdr", (texture) => {
  hdr = texture;
  texture.mapping = THREE.EquirectangularReflectionMapping;
  texture.premultiplyAlpha = false;
  scene.background = texture; // 배경
  scene.environment = texture; // 광원
});

function init() {
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(
    containerRef.value.offsetWidth,
    containerRef.value.offsetHeight,
  );
  containerRef.value.appendChild(renderer.domElement);
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update();
}

function animate() {
  camera.updateMatrixWorld();
  renderer.render(scene, camera);
  raf = requestAnimationFrame(animate);
}

const onResize = () => {
  camera.aspect =
    containerRef.value.offsetWidth / containerRef.value.offsetHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(
    containerRef.value.offsetWidth,
    containerRef.value.offsetHeight,
  );
};

onMounted(() => {
  camera = new THREE.PerspectiveCamera(
    80,
    containerRef.value.offsetWidth / containerRef.value.offsetHeight,
    0.1,
    1000,
  );
  camera.position.set(0, 0, 3);

  init();
  animate();

  window.addEventListener("resize", onResize);
});

onBeforeUnmount(() => {
  cancelAnimationFrame(raf);
  renderer.dispose();

  if (hdr) {
    hdr.dispose();
  }

  window.removeEventListener("resize", onResize);
});
</script>

<style lang="scss" scoped>
.wrapper {
  width: 100%;
  height: 100vh;
  .container {
    position: relative;
    width: 100%;
    height: 100%;
  }
}
</style>
