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

const addSphere = () => {
  const geometry = new THREE.SphereGeometry(0.5, 32, 32);
  const material = new THREE.MeshBasicMaterial({ color: 0xffffff });

  for (let z = -1000; z < 1000; z += 5) {
    const sphere = new THREE.Mesh(geometry, material);

    sphere.position.x = (Math.random() - 0.5) * 1000;
    sphere.position.y = (Math.random() - 0.5) * 1000;
    sphere.position.z = z;

    sphere.scale.set(2, 2, 2);

    group.add(sphere);
  }
};

let sampler;
const paths = [];
const loadObj = () => {
  new OBJLoader().load(
    "/whale.obj",
    (obj) => {
      sampler = new MeshSurfaceSampler(obj.children[0]).build();

      for (let i = 0; i < 4; i++) {
        const path = new Path(i);
        paths.push(path);
        group.add(path.line);
      }
    },
    // (xhr) => console.log((xhr.loaded / xhr.total) * 100 + "% loaded"),
  );
};

const tempPosition = new THREE.Vector3();
const materials = [
  new THREE.LineBasicMaterial({
    color: 0xd7e0ff,
    transparent: true,
    opacity: 0.5,
  }),
  new THREE.LineBasicMaterial({
    color: 0xe2e9ff,
    transparent: true,
    opacity: 0.5,
  }),
  new THREE.LineBasicMaterial({
    color: 0xa5b9ff,
    transparent: true,
    opacity: 0.5,
  }),
  new THREE.LineBasicMaterial({
    color: 0xb0c2ff,
    transparent: true,
    opacity: 0.5,
  }),
];
class Path {
  constructor(index) {
    this.geometry = new THREE.BufferGeometry();
    this.material = materials[index % 4];
    this.line = new THREE.Line(this.geometry, this.material);
    this.vertices = [];

    sampler.sample(tempPosition);
    this.previousPoint = tempPosition.clone();
  }

  update() {
    let pointFound = false;
    while (!pointFound) {
      sampler.sample(tempPosition);
      if (tempPosition.distanceTo(this.previousPoint) < 5) {
        this.vertices.push(tempPosition.x, tempPosition.y, tempPosition.z);
        this.previousPoint = tempPosition.clone();
        pointFound = true;
      }
    }
    this.geometry.setAttribute(
      "position",
      new THREE.Float32BufferAttribute(this.vertices, 3),
    );
  }
}

const animate = () => {
  group.rotation.y += 0.001;

  paths.forEach((path) => {
    if (path.vertices.length < 5000) {
      path.update();
    }
  });

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
  camera.position.z = 30;

  loadObj();
  addSphere();
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
  background: #111111;
}
</style>
