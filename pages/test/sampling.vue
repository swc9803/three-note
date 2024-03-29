<template>
  <div>
    <div ref="canvasRef" class="canvas" />
  </div>
</template>

<script setup>
import * as THREE from "three";
import { MeshSurfaceSampler } from "three/examples/jsm/math/MeshSurfaceSampler.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader.js";

const canvasRef = ref();
let camera;
let renderer;
let raf;

const scene = new THREE.Scene();

const group = new THREE.Group();
scene.add(group);

const vertices = [];
const colors = [];
const sparklesGeometry = new THREE.BufferGeometry();
const sparklesMaterial = new THREE.PointsMaterial({
  size: 3,
  alphaTest: 0.2,
  map: new THREE.TextureLoader().load(
    "https://assets.codepen.io/127738/dotTexture.png",
  ),
  vertexColors: true,
});
const points = new THREE.Points(sparklesGeometry, sparklesMaterial);
group.add(points);

let sampler;
let elephant;
new OBJLoader().load(
  "https://assets.codepen.io/127738/Mesh_Elephant.obj",
  (obj) => {
    elephant = obj.children[0];
    elephant.material = new THREE.MeshBasicMaterial({
      wireframe: true,
      color: 0x000000,
      transparent: true,
      opacity: 0.05,
    });
    group.add(obj);

    sampler = new MeshSurfaceSampler(elephant).build();
  },
  (xhr) => console.log((xhr.loaded / xhr.total) * 100 + "% loaded"),
  (err) => console.error(err),
);

const palette = [
  new THREE.Color("#FAAD80"),
  new THREE.Color("#FF6767"),
  new THREE.Color("#FF3D68"),
  new THREE.Color("#A73489"),
];
const tempPosition = new THREE.Vector3();

const addPoint = () => {
  sampler.sample(tempPosition);
  vertices.push(tempPosition.x, tempPosition.y, tempPosition.z);
  sparklesGeometry.setAttribute(
    "position",
    new THREE.Float32BufferAttribute(vertices, 3),
  );

  const color = palette[Math.floor(Math.random() * palette.length)];
  colors.push(color.r, color.g, color.b);
  sparklesGeometry.setAttribute(
    "color",
    new THREE.Float32BufferAttribute(colors, 3),
  );
};

const animate = () => {
  group.rotation.y += 0.002;

  if (sampler && vertices.length < 30000) {
    addPoint();
  }

  renderer.render(scene, camera);

  raf = requestAnimationFrame(animate);
};

const onResize = () => {
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
  canvasRef.value.appendChild(renderer.domElement);
  camera.aspect = canvasRef.value.offsetWidth / canvasRef.value.offsetHeight;
  camera.updateProjectionMatrix();
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.update();
};

onMounted(() => {
  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });

  camera = new THREE.PerspectiveCamera(
    75,
    canvasRef.value.offsetWidth / canvasRef.value.offsetHeight,
    0.1,
    1000,
  );
  camera.position.z = 300;

  onResize();
  animate();

  window.addEventListener("resize", onResize);
});

onBeforeUnmount(() => {
  cancelAnimationFrame(raf);

  window.removeEventListener("resize", onResize);
});
</script>

<style lang="scss" scoped>
.canvas {
  position: fixed;
  top: 0;
  left: 0;
  transform: translate3d(0, 0, 0);
  width: 100%;
  height: 100vh;
}
</style>
