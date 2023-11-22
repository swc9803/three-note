<template>
  <div class="wrapper">
    <div ref="containerRef" class="container" />
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

const containerRef = ref();
let camera;
let raf;

const scene = new THREE.Scene();
const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

const zoomFit = (object3d, camera) => {
  // 모델의 경계 박스
  const box = new THREE.Box3().setFromObject(object3d);

  // 모델의 경계 박스 대각 길이
  const sizeBox = box.getSize(new THREE.Vector3()).length();

  // 모델의 경계 박스 중심 위치
  const centerBox = box.getCenter(new THREE.Vector3());

  // 모델 크기의 절반값
  const halfSizeModel = sizeBox / 2;

  // 카메라의 fov의 절반값
  const halfFov = THREE.MathUtils.degToRad(camera.fov / 2);

  // 모델을 화면에 꽉 채우기 위한 거리
  const distance = halfSizeModel / Math.tan(halfFov);

  // 모델 중심에서 카메라 위치로 향하는 방향 단위 벡터 계산
  const direction = new THREE.Vector3()
    .subVectors(camera.position, centerBox)
    .normalize();

  console.log(direction);

  // 단위 방향 벡터 방향으로 모델 중심 위치에서 distance 거리에 대한 위치
  const position = direction.multiplyScalar(distance).add(centerBox);
  camera.position.copy(position);

  // 모델의 크기에 맞춰 카메라의 near, far 값 조정
  camera.near = sizeBox / 100;
  camera.far = sizeBox * 100;

  // 카메라 기본 속성 변경에 따른 투영행렬 업데이트
  camera.updateProjectionMatrix();

  // 카메라가 모델의 중심을 보게 함
  camera.lookAt(centerBox.x, centerBox.y, centerBox.z);
};

const gltfLoader = new GLTFLoader();
gltfLoader.load("/adamHead/adamHead.gltf", (model) => {
  scene.add(model.scene);
  zoomFit(model.scene, camera);
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

  scene.add(camera);

  // light
  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(1, 1, 1);
  camera.add(light);

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
