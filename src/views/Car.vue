<template>
  <div class="car-show">
    <!-- 用于渲染Three.js的canvas -->
    <div ref="sceneContainerRef" class="three-container"></div>
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { GUI } from "dat.gui";
import { ref, onMounted } from "vue";
let scene = null;
let camera = null;
let renderer = null;
let controls = null;
let  carMaterial = { color: "#6a6a70", metalness: 0.1, roughness: 0.1 };

const sceneContainerRef = ref(null);
onMounted(() => {
  initScene();
  loadCarModel();
  initControls();
  initGUI();
  animate();
});
function initScene() {
  // 初始化Three.js场景
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  camera.position.set(6.4, 3.4, 3);

  renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  sceneContainerRef.value.appendChild(renderer.domElement);

  const ambientLight = new THREE.AmbientLight(0xffffff, 1); // 提高环境光强度
  scene.add(ambientLight);

  const pointLight = new THREE.PointLight(0xffffff, 2); // 提高点光源强度
  pointLight.position.set(10, 10, 10);
  scene.add(pointLight);


  // 添加地板
  const planeGeometry = new THREE.PlaneGeometry(100, 100);
  const planeMaterial = new THREE.MeshStandardMaterial({ color:'#7b7c93' });
  const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  plane.rotation.x = -Math.PI / 2;
  plane.position.y = -0.5;
  scene.add(plane);
}
const carModel = ref(null);
function loadCarModel() {
  // 加载3D车模型
  const loader = new GLTFLoader();
  loader.load(
    'src/assets/scene.gltf',
    (gltf) => {
      debugger
      const car = gltf.scene;
    // 调整模型的位置和缩放比例
    car.position.set(0, 0, 0); // 移动到场景中心
    car.scale.set(1, 1, 1); // 缩放到合适大小
   
    if (car) {
      car.traverse((child) => {
        if (child.isMesh) {
          child.material.color.set(carMaterial.color);
          child.material.metalness = carMaterial.metalness;
          child.material.roughness = carMaterial.roughness;
        }
      });
    }
     carModel.value = car;
    scene.add(car); 
    },
    undefined,
    (error) => {
      console.error("Error loading model:", error);
    }
  );
}
function initControls() {
  // 初始化OrbitControls
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.8;
}
const gui = ref(null);
function initGUI() {
  // 初始化dat.GUI用于调节参数
  gui.value = new GUI();
  const cameraFolder = gui.value.addFolder("Camera");
  cameraFolder.add(camera.position, "x", -50, 50).name("X Position");
  cameraFolder.add(camera.position, "y", -50, 50).name("Y Position");
  cameraFolder.add(camera.position, "z", -50, 50).name("Z Position");
  cameraFolder.open();

  const materialFolder = gui.value.addFolder("Material");

  materialFolder.addColor(carMaterial, "color").onChange((value) => {
    if (carModel.value) {
      carModel.value.traverse((child) => {
        if (child.isMesh) {
          child.material.color.set(value);
        }
      });
    }
  });
  materialFolder.add(carMaterial, "metalness", 0, 1).onChange((value) => {
    if (carModel.value) {
      carModel.value.traverse((child) => {
        if (child.isMesh) {
          child.material.metalness = value;
        }
      });
    }
  });
  materialFolder.add(carMaterial, "roughness", 0, 1).onChange((value) => {
    if (carModel.value) {
      carModel.value.traverse((child) => {
        if (child.isMesh) {
          child.material.roughness = value;
        }
      });
    }
  });
  materialFolder.open();
}
function animate() {
    requestAnimationFrame(animate);// 请求动画帧
    controls.update();
    renderer.render(scene, camera);// 渲染场景
}


</script>

<style>
.car-show {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.three-container {
  width: 100%;
  height: 100%;
}
</style>
