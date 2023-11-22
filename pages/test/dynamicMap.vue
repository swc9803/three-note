<template>
  <div class="wrapper">
    <div ref="containerRef" class="container" />
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

const containerRef = ref();
let camera;
let raf;

const scene = new THREE.Scene();
const renderer = new THREE.WebGLRenderer({ antialias: true });

// light
scene.add(new THREE.AmbientLight(0xffffff, 0.2));

const color = 0xffffff;
const intensity = 5;
const light = new THREE.DirectionalLight(color, intensity);
light.position.set(-1, 2, 4);
scene.add(light);

// model
const renderTargetOptions = {
  format: THREE.RGBAFormat,
  generateMipmaps: true,
  minFilter: THREE.LinearMipmapLinearFilter,
};

const sphereRenderTarget = new THREE.WebGLCubeRenderTarget(
  512,
  renderTargetOptions,
);
const sphereCamera = new THREE.CubeCamera(0.1, 1000, sphereRenderTarget);
const sphereGeometry = new THREE.SphereGeometry(1.5);
const sphereMaterial = new THREE.MeshPhongMaterial({
  color: 0xffffff,
  envMap: sphereRenderTarget.texture,
  reflectivity: 0.95,
});
const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
const spherePivot = new THREE.Object3D();
spherePivot.add(sphere);
spherePivot.add(sphereCamera);
spherePivot.position.set(1, 0, 1);
scene.add(spherePivot);

const cylinderRenderTarget = new THREE.WebGLCubeRenderTarget(
  2048,
  renderTargetOptions,
);
const cylinderCamera = new THREE.CubeCamera(0.1, 1000, cylinderRenderTarget);
const cylinderGeometry = new THREE.CylinderGeometry(0.5, 1, 3, 32);
const cylinderMaterial = new THREE.MeshPhongMaterial({
  color: 0xffffff,
  envMap: cylinderRenderTarget.texture,
  reflectivity: 0.95,
});
const cylinder = new THREE.Mesh(cylinderGeometry, cylinderMaterial);
const cylinderPivot = new THREE.Object3D();
cylinderPivot.add(cylinder);
cylinderPivot.add(cylinderCamera);
cylinderPivot.position.set(-1, 0, -1);
scene.add(cylinderPivot);

const torusRenderTarget = new THREE.WebGLCubeRenderTarget(
  2048,
  renderTargetOptions,
);
const torusCamera = new THREE.CubeCamera(0.1, 1000, torusRenderTarget);
const torusGeometry = new THREE.TorusGeometry(4, 0.5, 24, 64);
const torusMaterial = new THREE.MeshPhongMaterial({
  color: 0xffffff,
  envMap: torusRenderTarget.texture,
  reflectivity: 0.95,
});
const torus = new THREE.Mesh(torusGeometry, torusMaterial);
const torusPivot = new THREE.Object3D();
torusPivot.add(torus);
torusPivot.add(torusCamera);
torus.rotation.x = Math.PI / 2;
scene.add(torusPivot);
torus.name = "torus";

const planeRenderTarget = new THREE.WebGLCubeRenderTarget(
  2048,
  renderTargetOptions,
);
const planeCamera = new THREE.CubeCamera(0.1, 1000, planeRenderTarget);
const planeGeometry = new THREE.PlaneGeometry(12, 12);
const planeMaterial = new THREE.MeshPhongMaterial({
  color: 0xffffff,
  envMap: planeRenderTarget.texture,
  reflectivity: 0.95,
});
const plane = new THREE.Mesh(planeGeometry, planeMaterial);
const planePivot = new THREE.Object3D();
planePivot.add(plane);
planePivot.add(planeCamera);
plane.rotation.x = -Math.PI / 2;
planePivot.position.y = -4.8;
scene.add(planePivot);

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

let time = 0;
function animate() {
  camera.updateMatrixWorld();
  renderer.render(scene, camera);

  scene.traverse((obj) => {
    if (obj instanceof THREE.Object3D) {
      const mesh = obj.children[0];
      const cubeCamera = obj.children[1];
      if (
        mesh instanceof THREE.Mesh &&
        cubeCamera instanceof THREE.CubeCamera
      ) {
        mesh.visible = false;
        cubeCamera.update(renderer, scene);
        mesh.visible = true;
      }
    }
  });

  time += 0.01;
  torus.rotation.x = Math.sin(time);

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
  camera.position.set(0, 0, 10);

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
