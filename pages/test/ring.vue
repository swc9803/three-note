<template>
  <div class="container">
    <div ref="canvasRef" class="canvas" />
    <div class="btnWrapper">
      <div>
        <div
          v-for="color in ringColors"
          :key="color.id"
          :ref="ringColorsRef"
          class="ringColor"
        />
      </div>
      <div>
        <div
          v-for="color in gemColors"
          :key="color.id"
          :ref="gemColorsRef"
          class="gemColor"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import gsap from "gsap";

// light
const lights = [];

const canvasRef = ref();
const ringColorArray = ref([]);
const ringColorsRef = (el) => ringColorArray.value.push(el);
const gemColorArray = ref([]);
const gemColorsRef = (el) => gemColorArray.value.push(el);

const ringColors = ["#D4AF37", "#8A8D8F", "#E6C7C2"];
const gemColors = [
  "#6ab04c",
  "#ffffff",
  "#eb4d4b",
  "#f0932b",
  "#130f40",
  "#111111",
];

let renderer;
let scene;
let camera;
let raf;

let ringModel;
const setupModel = () => {
  new RGBELoader().load("/brown_photostudio_03_4k.hdr", (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.background = texture;
    scene.environment = texture;
    // scene.backgroundBlurriness = 0.03;

    const goldMaterial = new THREE.MeshStandardMaterial({
      color: new THREE.Color("#D4AF37"),
      envMap: texture,
      envMapIntensity: 1,
      metalness: 1,
      roughness: 0.15,
    });

    const gemMaterial = new THREE.MeshPhysicalMaterial({
      color: new THREE.Color("#6AB04C"),
      metalness: 0.5,
      roughness: 0,
      opacity: 0.8,
      side: THREE.DoubleSide,
      transparent: true,
      transmission: 0.5,
      ior: 2.4,
      thickness: 0.3,
    });

    const loader = new GLTFLoader();
    loader.load("/ring.glb", (gltf) => {
      ringModel = gltf.scene;
      ringModel.scale.set(0.3, 0.3, 0.3);
      scene.add(ringModel);

      ringModel.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          if (child.name.startsWith("GEM")) {
            child.material = gemMaterial;
          } else {
            child.material = goldMaterial;
          }
        }
      });
    });
  });
};

const setupUi = () => {
  for (let i = 0; i < ringColorArray.value.length; i++) {
    const color = ringColors[i];
    ringColorArray.value[i].style.backgroundColor = color;
    ringColorArray.value[i].setAttribute("color", color);

    ringColorArray.value[i].addEventListener("click", (e) => {
      const color = e.target.getAttribute("color");
      ringModel.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          if (!child.name.startsWith("GEM")) {
            const materialColor = child.material.color;
            const newColor = new THREE.Color(color);
            gsap.to(materialColor, {
              r: newColor.r,
              g: newColor.g,
              b: newColor.b,
              duration: 0.5,
            });
          }
        }
      });
    });
  }

  for (let i = 0; i < gemColorArray.value.length; i++) {
    const color = gemColors[i];
    gemColorArray.value[i].style.backgroundColor = color;
    gemColorArray.value[i].setAttribute("color", color);

    gemColorArray.value[i].addEventListener("click", (e) => {
      const color = e.target.getAttribute("color");
      ringModel.traverse((child) => {
        if (child instanceof THREE.Mesh) {
          if (child.name.startsWith("GEM")) {
            const materialColor = child.material.color;
            const newColor = new THREE.Color(color);
            gsap.to(materialColor, {
              r: newColor.r,
              g: newColor.g,
              b: newColor.b,
              duration: 0.5,
            });
          }
        }
      });
    });
  }
};

let controls;
function init() {
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(canvasRef.value.offsetWidth, canvasRef.value.offsetHeight);
  canvasRef.value.appendChild(renderer.domElement);

  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.maxDistance = 4;
}

let time = 0;
function animate() {
  camera.updateMatrixWorld();
  renderer.render(scene, camera);
  controls.update();

  time += 0.005;
  if (lights) {
    const r = 3;
    const gap = THREE.MathUtils.degToRad(360 / lights.length);
    for (let i = 0; i < lights.length; i++) {
      const light = lights[i];
      light.position.set(
        gap * i,
        Math.cos(time + gap * i) * r,
        Math.sin(time + gap * i) * r,
      );
    }
  }
  if (ringModel) ringModel.rotation.z = time;

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
  renderer.outputColorSpace = THREE.SRGBColorSpace;
  renderer.toneMappingExposure = 2.5;

  for (let i = 0; i < 4; i++) {
    const spotLight = new THREE.SpotLight(0xffffff, 0.5);
    lights.push(spotLight);
    scene.add(spotLight);
  }

  camera = new THREE.PerspectiveCamera(
    70,
    canvasRef.value.offsetWidth / canvasRef.value.offsetHeight,
    0.1,
    10,
  );
  camera.position.set(-0.3, 0.25, -0.5);

  setupModel();
  setupUi();

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
  .btnWrapper {
    position: absolute;
    left: 50%;
    bottom: 3%;
    transform: translate(-50%, 0);
    display: flex;
    gap: 3em;
    div {
      display: flex;
      align-items: center;
      gap: 0.4em;
      .ringColor {
        width: 6em;
        height: 6em;
        border: 0.4em solid rgba(0, 0, 0, 0.5);
        border-radius: 50%;
        cursor: pointer;
      }
      .gemColor {
        width: 5.5em;
        height: 5.5em;
        border: 0.4em solid rgba(0, 0, 0, 0.5);
        border-radius: 30%;
        cursor: pointer;
      }
    }
  }
}
</style>
