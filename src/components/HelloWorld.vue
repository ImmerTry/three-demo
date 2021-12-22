<script setup>
import { ref, toRefs, onMounted, reactive } from "vue";
import {
  Scene,
  Color,
  WebGLRenderer,
  PerspectiveCamera,
  DirectionalLight,
  DirectionalLightHelper,
  HemisphereLight,
  HemisphereLightHelper,
} from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

// 引入模型的loader实例
const loader = new GLTFLoader();
let scene,
  renderer,
  camera,
  directionalLight,
  hemisphereLight,
  dhelper,
  hHelper,
  controls;

let loadingWidth = ref(0);
let isLoading = ref(true);
// 相机默认坐标
let defaultMap = {
  x: 510,
  y: 128,
  z: 0,
};
const map = reactive(defaultMap);
const { x, y, z } = toRefs(map);

//车身颜色数组
const colorAry = [
  "rgb(216, 27, 67)",
  "rgb(142, 36, 170)",
  "rgb(81, 45, 168)",
  "rgb(48, 63, 159)",
  "rgb(30, 136, 229)",
  "rgb(0, 137, 123)",
  "rgb(67, 160, 71)",
  "rgb(251, 192, 45)",
  "rgb(245, 124, 0)",
  "rgb(230, 74, 25)",
  "rgb(233, 30, 78)",
  "rgb(156, 39, 176)",
  "rgb(0, 0, 0)",
];
// 1- 设置场景
const setScene = () => {
  scene = new Scene();
  renderer = new WebGLRenderer();
  renderer.setSize(innerWidth, innerHeight);
  document.querySelector(".boxs").appendChild(renderer.domElement);
};
// 2- 创建相机
const setCamera = () => {
  const { x, y, z } = defaultMap;
  camera = new PerspectiveCamera(60, innerWidth / innerHeight, 1, 1000);
  camera.position.set(x, y, z);
};
// 3-加载3D模型文件
const loadFile = (url) => {
  return new Promise((resolve, reject) => {
    loader.load(
      url,
      (gltf) => {
        resolve(gltf);
      },
      ({ loaded, total }) => {
        let load = Math.abs((loaded / total) * 100);
        loadingWidth.value = load;
        if (load >= 100) {
          setTimeout(() => {
            isLoading.value = false;
          }, 1000);
        }
        console.log((loaded / total) * 100 + "% loaded");
      },
      (err) => {
        reject(err);
      }
    );
  });
};
// 设置灯光
const setLight = () => {
  directionalLight = new DirectionalLight(0xffffff, 0.5);
  directionalLight.position.set(-4, 8, 4);
  dhelper = new DirectionalLightHelper(directionalLight, 5, 0xffffff);

  hemisphereLight = new HemisphereLight(0xffffff, 0xffffff, 0.4);
  hemisphereLight.position.set(0, 8, 5);
  hHelper = new HemisphereLightHelper(hemisphereLight, 5);

  scene.add(directionalLight);
  scene.add(hemisphereLight);
};
//使场景、照相机、模型不停调用
const loop = () => {
  requestAnimationFrame(loop);
  renderer.render(scene, camera);
  controls.update();
};
// 使用鼠标进行控制
const setControls = () => {
  controls = new OrbitControls(camera, renderer.domElement);
  controls.maxPolarAngle = (0.9 * Math.PI) / 2;
  controls.enableZoom = true;
  controls.addEventListener("change", render);
};
// 坐标信息
const render = () => {
  map.x = Number.parseInt(camera.position.x);
  map.y = Number.parseInt(camera.position.y);
  map.z = Number.parseInt(camera.position.z);
};
//是否自动转动
const isAutoFun = () => {
  controls.autoRotate = true;
};
//停止转动
const stop = () => {
  controls.autoRotate = false;
};
// 设置车身颜色
const setCarColor = (index) => {
  const currentColor = new Color(colorAry[index]);
  scene.traverse((child) => {
    if (child.isMesh) {
      // console.log(child.name);
      if (child.name.includes("door_")) {
        child.material.color.set(currentColor);
      }
    }
  });
};
// 页面初始化
const init = async () => {
  setScene();
  setCamera();
  setLight();
  setControls();
  const gltf = await loadFile("src/assets/3d/tesla_2018_model_3/scene.gltf");

  scene.add(gltf.scene);
  loop();
};
// 钩子函数
onMounted(init);
</script>

<template>
  <div class="boxs">
    <div class="maskLoading" v-if="isLoading">
      <div class="loading">
        <div :style="{ width: loadingWidth + '%' }"></div>
      </div>
      <div style="padding-left: 10px">{{ parseInt(loadingWidth) }}%</div>
    </div>
    <div class="mask">
      <p>x : {{ x }} y:{{ y }} z :{{ z }}</p>
      <button @click="isAutoFun">转动车</button>
      <button @click="stop">停止</button>
      <div class="flex">
        <div
          @click="setCarColor(index)"
          v-for="(item, index) in colorAry"
          :key="index"
          :style="{ backgroundColor: item }"
        ></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
body {
  margin: 0;
}

.maskLoading {
  background: #000;
  position: fixed;
  display: flex;
  justify-content: center;
  align-items: center;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  z-index: 1111111;
  color: #fff;
}

.maskLoading .loading {
  width: 400px;
  height: 20px;
  border: 1px solid #fff;
  background: #000;
  overflow: hidden;
  border-radius: 10px;
}

.maskLoading .loading div {
  background: #fff;
  height: 20px;
  width: 0;
  transition-duration: 500ms;
  transition-timing-function: ease-in;
}

canvas {
  width: 100%;
  height: 100%;
  margin: auto;
}

.mask {
  color: #fff;
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
}

.flex {
  display: flex;
  flex-wrap: wrap;
  padding: 20px;
}

.flex div {
  width: 10px;
  height: 10px;
  margin: 5px;
  cursor: pointer;
}
</style>
