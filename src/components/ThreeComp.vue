<template>
  <div class="three-wrap" ref="canvasWrap"></div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
const canvasWrap = ref(null)

import * as THREE from 'three'
import WebGL from 'three/addons/capabilities/WebGL.js'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { PointerLockControls } from 'three/examples/jsm/controls/PointerLockControls'

// 创建场景
const scene = new THREE.Scene()

let camera
let renderer
// 控制器
let controls

const init = () => {
  // 创建照相机
  camera = new THREE.PerspectiveCamera(
    75,
    canvasWrap.value.clientWidth / canvasWrap.value.clientHeight,
    0.1,
    1000
  )
  // 创建渲染器
  renderer = new THREE.WebGLRenderer()
  renderer.setSize(canvasWrap.value.clientWidth, canvasWrap.value.clientHeight)
  // 渲染器背景
  renderer.setClearColor(0x000000, 1)
  canvasWrap.value.appendChild(renderer.domElement)

  // 创建控制器
  controls = new OrbitControls(camera, renderer.domElement)
  // controls = new PointerLockControls(camera, renderer.domElement)
  // renderer.domElement.addEventListener('click', () => {
  //   controls.lock()
  // })
}

// 视口窗口大小变化时，重新渲染
const resize = () => {
  camera.aspect = canvasWrap.value.clientWidth / canvasWrap.value.clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(canvasWrap.value.clientWidth, canvasWrap.value.clientHeight)
}
// 创建一个点
let points = [
  { position: new THREE.Vector3(0, 0, 0), element: null },
  { position: new THREE.Vector3(1, 1, 1), element: null },
  { position: new THREE.Vector3(1, 1, -1), element: null },
  { position: new THREE.Vector3(1, -1, 1), element: null },
  { position: new THREE.Vector3(1, -1, -1), element: null },
  { position: new THREE.Vector3(-1, 1, 1), element: null },
  { position: new THREE.Vector3(-1, 1, -1), element: null },
  { position: new THREE.Vector3(-1, -1, 1), element: null },
  { position: new THREE.Vector3(-1, -1, -1), element: null }
]
const raycaster = new THREE.Raycaster()
// 创建
const create = () => {
  const geometry = new THREE.BoxGeometry()
  const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
  const cube = new THREE.Mesh(geometry, material)
  scene.add(cube)
  camera.position.z = 5

  createPoints()
}

// 创建所有点
const createPoints = () => {
  points.forEach((point) => {
    const element = document.createElement('div')
    element.classList.add('point')
    canvasWrap.value.appendChild(element)
    point.element = element
  })
}

const animate = () => {
  requestAnimationFrame(animate)
  renderer.render(scene, camera)

  // 更新点的位置
  points.forEach((point) => {
    const screenPosition = point.position.clone()
    // 通过射线判断点是否在被遮挡
    raycaster.setFromCamera(screenPosition, camera)
    screenPosition.project(camera)
    const intersects = raycaster.intersectObjects(scene.children)
    if (intersects.length === 0) {
      // 未找到相交点，显示
      point.element.classList.add('visible')
    } else {
      // 找到相交点
      // 获取相交点的距离和点的距离
      const intersectionDistance = intersects[0].distance
      const pointDistance = point.position.distanceTo(camera.position)
      // 相交点距离比点距离近，隐藏；相交点距离比点距离远，显示
      intersectionDistance < pointDistance
        ? point.element.classList.remove('visible')
        : point.element.classList.add('visible')
    }

    const x = ((screenPosition.x + 1) / 2) * canvasWrap.value.clientWidth
    const y = (-(screenPosition.y - 1) / 2) * canvasWrap.value.clientHeight
    point.element.style.left = x + 'px'
    point.element.style.top = y + 'px'
  })
}

onMounted(() => {
  init()
  create()
  nextTick(() => {
    animate()
  })
  const observer = new ResizeObserver(resize)
  observer.observe(canvasWrap.value)
})
</script>

<style lang="scss" scoped>
.three-wrap {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  ::v-deep(.point) {
    position: absolute;
    width: 10px;
    height: 10px;
    background-color: red;
    transform: translate(-50%, -50%);
    z-index: 100;
    transition: all 0.5s ease-out;
    display: block;
    &.hide {
      display: none;
    }
  }
}
</style>
