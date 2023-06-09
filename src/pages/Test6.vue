<template>
	<div class="wrapper">
		<div ref="containerRef" class="container" />
	</div>
</template>

<script setup>
import * as THREE from 'three';
import gsap from 'gsap';

const containerRef = ref();

onMounted(() => {
	const radians = degrees => (degrees * Math.PI) / 180;
	const distance = (x1, y1, x2, y2) =>
		Math.sqrt(Math.pow(x1 - x2, 2) + Math.pow(y1 - y2, 2));
	const map = (value, start1, stop1, start2, stop2) => {
		return (
			((value - start1) / (stop1 - start1)) * (stop2 - start2) + start2
		);
	};
	class LilShape {
		constructor() {
			this.geom = new THREE.BoxGeometry(0.6, 0.6, 0.6);
			this.rotationX = 0;
			this.rotationY = 0;
			this.rotationZ = 0;
		}
	}
	class Cube extends LilShape {
		constructor() {
			super(new LilShape());
			this.rotationX = radians(-45);
			this.rotationY = radians(45);
		}
	}
	class Torus extends LilShape {
		constructor() {
			super(new LilShape());
			this.geom = new THREE.TorusGeometry(0.3, 0.12, 30, 200);
			this.rotationX = radians(90);
		}
	}
	class Icosahedron extends LilShape {
		constructor() {
			super(new LilShape());
			this.geom = new THREE.IcosahedronGeometry(0.45);
		}
	}
	class Cone extends LilShape {
		constructor() {
			super(new LilShape());
			this.geom = new THREE.ConeGeometry(0.4, 1, 30);
			this.rotationX = radians(90);
		}
	}
	class Dodecahedron extends LilShape {
		constructor() {
			super(new LilShape());
			this.geom = new THREE.DodecahedronGeometry(0.5, 0);
		}
	}
	class App {
		setup() {
			this.gutter = {
				size: 4,
			};
			this.meshes = [];
			this.grid = {
				rows: 30,
				cols: 30,
			};
			this.width = window.innerWidth;
			this.height = window.innerHeight;
			this.mouse3D = new THREE.Vector2();
			this.geometries = [
				new Cube(),
				new Torus(),
				new Icosahedron(),
				new Cone(),
				new Dodecahedron(),
			];

			this.raycaster = new THREE.Raycaster();
		}

		createScene() {
			this.scene = new THREE.Scene();

			this.renderer = new THREE.WebGLRenderer({
				antialias: true,
				alpha: true,
			});
			this.renderer.setSize(this.width, this.height);
			this.renderer.setPixelRatio(window.devicePixelRatio);

			this.renderer.shadowMap.enabled = true;
			this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
			containerRef.value.appendChild(this.renderer.domElement);
		}

		createCamera() {
			this.camera = new THREE.PerspectiveCamera(
				20,
				window.innerWidth / window.innerHeight,
				1,
			);
			this.camera.position.set(0, 65, 0);
			this.camera.rotation.x = -1.57;

			this.scene.add(this.camera);
		}

		addAmbientLight() {
			const light = new THREE.AmbientLight('#ffffff', 1);

			this.scene.add(light);
		}

		addSpotLight() {
			const light = new THREE.SpotLight('#FFF', 1, 1000);
			light.position.set(0, 27, 0);
			this.scene.add(light);
		}

		addPointLight(color, position) {
			const light = new THREE.PointLight(color, 1, 1000, 1);

			light.position.set(position.x, position.y, position.z);

			this.scene.add(light);
		}

		getRandomGeometry() {
			return this.geometries[
				Math.floor(Math.random() * Math.floor(this.geometries.length))
			];
		}

		createGrid() {
			this.groupMesh = new THREE.Object3D();
			const materials = [
				new THREE.MeshPhysicalMaterial({
					color: '#E2A9E5',
					metalness: 0.58,
					emissive: '#000000',
					roughness: 0.18,
				}),
				new THREE.MeshStandardMaterial({
					color: '#E2A9E5',
					metalness: 1,
					roughness: 0.5,
					aoMapIntensity: 1,
					envMapIntensity: 1,
					displacementScale: 2.4,
					normalScale: 1,
				}),
			];

			for (let row = 0; row < this.grid.rows; row++) {
				this.meshes[row] = [];

				for (let index = 0; index < 1; index++) {
					const totalCol = this.getTotalRows(row);

					for (let col = 0; col < totalCol; col++) {
						const geometry = this.getRandomGeometry();
						const mesh = this.getMesh(
							geometry.geom,
							materials[
								Math.floor(Math.random() * materials.length)
							],
						);

						mesh.position.y = 0;
						mesh.position.x =
							col +
							col * this.gutter.size +
							(totalCol === this.grid.cols ? 0 : 2.5);
						mesh.position.z = row + row * (index + 0.25);

						mesh.rotation.x = geometry.rotationX;
						mesh.rotation.y = geometry.rotationY;
						mesh.rotation.z = geometry.rotationZ;

						mesh.initialRotation = {
							x: mesh.rotation.x,
							y: mesh.rotation.y,
							z: mesh.rotation.z,
						};

						this.groupMesh.add(mesh);

						this.meshes[row][col] = mesh;
					}
				}
			}

			const centerX = -(this.grid.cols / 2) * this.gutter.size - 1;
			const centerZ = -(this.grid.rows / 2) - 0.8;

			this.groupMesh.position.set(centerX, 0, centerZ);

			this.scene.add(this.groupMesh);
		}

		getTotalRows(col) {
			return col % 2 === 0 ? this.grid.cols : this.grid.cols - 1;
		}

		getMesh(geometry, material) {
			const mesh = new THREE.Mesh(geometry, material);

			mesh.castShadow = true;
			mesh.receiveShadow = true;

			return mesh;
		}

		draw() {
			this.raycaster.setFromCamera(this.mouse3D, this.camera);

			const intersects = this.raycaster.intersectObjects(
				this.scene.children,
			);

			if (intersects.length) {
				const { x, z } = intersects[0].point;

				for (let row = 0; row < this.grid.rows; row++) {
					for (let index = 0; index < 1; index++) {
						const totalCols = this.getTotalRows(row);

						for (let col = 0; col < totalCols; col++) {
							const mesh = this.meshes[row][col];
							mesh.initialRotation.z += radians(0.5);
							mesh.initialRotation.y -= radians(0.1);

							const mouseDistance = distance(
								x,
								z,
								mesh.position.x + this.groupMesh.position.x,
								mesh.position.z + this.groupMesh.position.z,
							);

							const y = map(mouseDistance, 5, 0, 0, 6);
							gsap.to(mesh.position, 0.3, {
								y: y < 1 ? 1 : y,
							});

							const scaleFactor = mesh.position.y / 1.7;
							const scale = scaleFactor < 1 ? 1 : scaleFactor;
							gsap.to(mesh.scale, 0.3, {
								ease: 'expo.easeOut',
								x: scale,
								y: scale,
								z: scale,
							});

							gsap.to(mesh.rotation, 2, {
								ease: 'quad.easeOut',
								x: map(
									mesh.position.y,
									-1,
									1,
									radians(180),
									mesh.initialRotation.x,
								),
								z: map(
									mesh.position.y,
									-1,
									1,
									radians(-90),
									mesh.initialRotation.z,
								),
								y: map(
									mesh.position.y,
									-1,
									1,
									radians(45),
									mesh.initialRotation.y,
								),
							});
						}
					}
				}
			}
		}

		init() {
			this.setup();
			this.createScene();
			this.createCamera();
			this.createGrid();
			this.addAmbientLight();
			this.addSpotLight();
			this.addPointLight(0xe2a9e5, {
				x: 0,
				y: 10,
				z: -100,
			});
			this.addPointLight(0x632c65, {
				x: 100,
				y: 10,
				z: 0,
			});
			this.addPointLight(0x4b384c, {
				x: 20,
				y: 5,
				z: 20,
			});
			this.animate();

			window.addEventListener('resize', this.onResize.bind(this));

			window.addEventListener(
				'mousemove',
				this.onMouseMove.bind(this),
				false,
			);

			this.onMouseMove({
				clientX: 0,
				clientY: 0,
			});
		}

		onMouseMove({ clientX, clientY }) {
			this.mouse3D.x = (clientX / this.width) * 2 - 1;
			this.mouse3D.y = -(clientY / this.height) * 2 + 1;
		}

		onResize() {
			this.width = window.innerWidth;
			this.height = window.innerHeight;

			this.camera.aspect = this.width / this.height;
			this.camera.updateProjectionMatrix();
			this.renderer.setSize(this.width, this.height);
		}

		animate() {
			this.draw();

			this.renderer.render(this.scene, this.camera);

			requestAnimationFrame(this.animate.bind(this));
		}
	}
	new App().init();
});
</script>

<style lang="scss" scoped>
.wrapper {
	width: 100%;
	height: calc(var(--vh) * 100);
	.container {
		position: relative;
		width: 100%;
		height: 100%;
	}
}
</style>
