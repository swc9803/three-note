<template>
  <div class="wrapper">
    <div ref="containerRef" class="container" />
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { TeapotGeometry } from "three/examples/jsm/geometries/TeapotGeometry.js";

const containerRef = ref();
let camera;
let raf;

const scene = new THREE.Scene();
const renderer = new THREE.WebGLRenderer({ antialias: true });

const loader = new THREE.TextureLoader();
loader.load("/christmas.jpg", (texture) => {
  const renderTarget = new THREE.WebGLCubeRenderTarget(texture.image.height);
  renderTarget.fromEquirectangularTexture(renderer, texture);
  scene.background = renderTarget.texture;
});

// light
const light = new THREE.DirectionalLight(0xffffff, 1);
light.position.set(1, 1, 1);
scene.add(light);

// teapot
const teapotRenderTarget = new THREE.WebGLCubeRenderTarget(1024, {
  format: THREE.RGBAFormat,
  generateMipmaps: true,
  minFilter: THREE.LinearMipmapLinearFilter,
});
teapotRenderTarget._pmremGen = new THREE.PMREMGenerator(renderer);

const teapotCamera = new THREE.CubeCamera(0.01, 10, teapotRenderTarget);

const teapotGeometry = new TeapotGeometry(0.7, 24);
const teapotMaterial = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,
  metalness: 0.1,
  roughness: 0.05,
  ior: 2.5, // 굴절률
  thickness: 0.2, // 유리 두께
  transmission: 1, // 투명도
  side: THREE.DoubleSide,
  envMap: teapotRenderTarget.texture,
  envMapIntensity: 1,
});
const teapot = new THREE.Mesh(teapotGeometry, teapotMaterial);
teapot.add(teapotCamera);
scene.add(teapot);

// cylinder
const cylinderGeometry = new THREE.CylinderGeometry(0.1, 0.2, 1.5, 32);
const cylinderMaterial = new THREE.MeshNormalMaterial();
const cylinderPivot = new THREE.Object3D();
for (let degree = 0; degree <= 360; degree += 30) {
  const cylinder = new THREE.Mesh(cylinderGeometry, cylinderMaterial);
  const radian = THREE.MathUtils.degToRad(degree);
  cylinder.position.set(2 * Math.sin(radian), 0, 2 * Math.cos(radian));
  cylinderPivot.add(cylinder);
}
scene.add(cylinderPivot);

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

  time += 0.001;
  if (cylinderPivot) {
    cylinderPivot.rotation.y = Math.sin(time);
  }

  // if (teapot) {
  // 	teapot.visible = false;

  // 	const teapotCamera = teapot.children[0];
  // 	teapotCamera.update(renderer, scene);

  // 	const pmremGen = teapotCamera.renderTarget._pmremGen;
  // 	const renderTarget = pmremGen.fromCubemap(
  // 		teapotCamera.renderTarget.texture,
  // 	);
  // 	pmremGen.dispose();

  // 	teapot.material.envMap = renderTarget.texture;
  // 	teapot.material.needsUpdate = true;

  // 	teapot.visible = true;
  // }

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
  camera.position.set(0, 0, 4);

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
