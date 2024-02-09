<template>
  <div class="container">
    <div ref="canvasRef" class="wrapper" />
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

const canvasRef = ref();
let camera;
let raf;

const fogColor = 0x004fff;
const objColor = 0xffffff;
const floorColor = 0x555555;

const scene = new THREE.Scene();
const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
scene.background = new THREE.Color(fogColor);
scene.fog = new THREE.Fog(fogColor, 2, 16); // 거리
// scene.fog = new THREE.FogExp2(fogColor, 0.2); // 밀도

// torus
const geometry = new THREE.TorusGeometry(0.7, 0.3, 12, 80);
const material = new THREE.MeshStandardMaterial({ color: objColor });
const obj = new THREE.Mesh(geometry, material);
obj.position.set(0, 0.8, 0);
scene.add(obj);

// plane
const planeGeometry = new THREE.PlaneGeometry(30, 30, 1, 1);
const planeMaterial = new THREE.MeshStandardMaterial({ color: floorColor });
const plane = new THREE.Mesh(planeGeometry, planeMaterial);
plane.rotation.set(-0.5 * Math.PI, 0, 0);
plane.position.set(0, -0.5, 0);
scene.add(plane);

// light
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(1, 1, 1);
scene.add(light);

const init = () => {
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
  canvasRef.value.appendChild(renderer.domElement);
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.minDistance = 3;
  controls.maxDistance = 6;
  controls.maxPolarAngle = Math.PI / 2 - 0.1;
  controls.update();
};

const animate = () => {
  camera.updateMatrixWorld();
  obj.rotation.y += 0.01;
  renderer.render(scene, camera);
  raf = requestAnimationFrame(animate);
};

const onResize = () => {
  camera.aspect = canvasRef.value.offsetWidth / canvasRef.value.offsetHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
};

onMounted(() => {
  camera = new THREE.PerspectiveCamera(
    75,
    canvasRef.value.offsetWidth / canvasRef.value.offsetHeight,
    0.1,
    10,
  );
  camera.position.set(0, 2, 3);
  camera.lookAt(0, 0, 0);

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
  .wrapper {
    width: 100%;
    height: 100%;
  }
}
</style>
