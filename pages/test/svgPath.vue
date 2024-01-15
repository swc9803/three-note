<template>
  <div class="container">
    <div ref="canvasRef" class="canvas" />
    <svg viewBox="0 0 600 552">
      <path
        d="M300,107.77C284.68,55.67,239.76,0,162.31,0,64.83,0,0,82.08,0,171.71c0,.48,0,.95,0,1.43-.52,19.5,0,217.94,299.87,379.69v0l0,0,.05,0,0,0,0,0v0C600,391.08,600.48,192.64,600,173.14c0-.48,0-.95,0-1.43C600,82.08,535.17,0,437.69,0,360.24,0,315.32,55.67,300,107.77"
        fill="#ee5282"
      />
    </svg>
  </div>
</template>

<script setup>
import gsap from "gsap";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

const canvasRef = ref();

let renderer;
let scene;
let camera;
let raf;

let controls;

const tl = gsap.timeline({
  repeat: -1,
  yoyo: true,
});

const path = document.querySelector("path");
const length = path.getTotalLength();
const vertices = [];
for (let i = 0; i < length; i += 0.1) {
  const point = path.getPointAtLength(i);
  const vector = new THREE.Vector3(point.x, -point.y, 0);
  vector.x += (Math.random() - 0.5) * 30;
  vector.y += (Math.random() - 0.5) * 30;
  vector.z += (Math.random() - 0.5) * 70;
  vertices.push(vector);
  // Create a tween for that vector
  tl.from(
    vector,
    {
      x: 600 / 2, // Center X of the heart
      y: -552 / 2, // Center Y of the heart
      z: 0, // Center of the scene
      ease: "power2.inOut",
      duration: "random(2, 5)", // Random duration
    },
    i * 0.002, // Delay calculated from the distance along the path
  );
}
const geometry = new THREE.BufferGeometry().setFromPoints(vertices);
const material = new THREE.PointsMaterial({
  color: 0xee5282,
  blending: THREE.AdditiveBlending,
  size: 3,
});
const particles = new THREE.Points(geometry, material);
// Offset the particles in the scene based on the viewbox values
particles.position.x -= 600 / 2;
particles.position.y += 552 / 2;
scene.add(particles);

gsap.fromTo(
  scene.rotation,
  {
    y: -0.2,
  },
  {
    y: 0.2,
    repeat: -1,
    yoyo: true,
    ease: "power2.inOut",
    duration: 3,
  },
);

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
.controls {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
}

canvas {
  position: absolute;
}
svg {
  position: absolute;
  display: none;
}
</style>
