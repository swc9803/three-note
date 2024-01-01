<template>
  <div class="wrapper">
    <div ref="canvasRef" class="container" />
  </div>
</template>

<script setup>
import * as THREE from "three";

const canvasRef = ref();
let camera;

const scene = new THREE.Scene();
const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

const addPlanet = () => {
  const geometry = new THREE.TorusGeometry(0.7, 0.3, 12, 80);
  const material = new THREE.MeshStandardMaterial({ color: 0xff0000 });
  const obj = new THREE.Mesh(geometry, material);
  scene.add(obj);
};

// light
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(1, 1, 1);
scene.add(light);

const onResize = () => {
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
  canvasRef.value.appendChild(renderer.domElement);
  camera.aspect = canvasRef.value.offsetWidth / canvasRef.value.offsetHeight;
  camera.updateProjectionMatrix();
};

const animate = () => {
  renderer.render(scene, camera);

  requestAnimationFrame(animate);
};

onMounted(() => {
  camera = new THREE.PerspectiveCamera(
    80,
    canvasRef.value.offsetWidth / canvasRef.value.offsetHeight,
    0.1,
    1000,
  );
  camera.position.set(0, 2, 3);
  camera.lookAt(0, 0, 0);

  onResize();
  animate();
  addPlanet();

  window.addEventListener("resize", onResize);
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
