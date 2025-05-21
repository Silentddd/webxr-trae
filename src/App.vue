<template>
  <el-container>
    <el-header>
      <h1>WebXR元宇宙体验</h1>
    </el-header>
    <el-main>
      <div id="xr-scene"></div>
      <el-button @click="enterVR">进入VR模式</el-button>
    </el-main>
  </el-container>
</template>

<script setup>
import { onMounted } from 'vue'
import * as THREE from 'three'
import TWEEN from '@tweenjs/tween.js'

let scene, camera, renderer, cube

function initScene() {
  console.log('开始初始化Three.js场景')
  // 创建Three.js场景
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x333333)
  console.log('场景创建完成')
  
  // 创建相机
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 5
  
  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true, xrCompatible: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  document.getElementById('xr-scene').appendChild(renderer.domElement)
  console.log('渲染器已挂载到DOM')
  
  // 添加WebXR支持
  if ('xr' in navigator) {
    navigator.xr.isSessionSupported('immersive-vr').then(supported => {
      if (supported) {
        renderer.xr.enabled = true
      }
    })
  }
  
  // 添加立方体
  const geometry = new THREE.BoxGeometry()
  const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 })
  cube = new THREE.Mesh(geometry, material)
  scene.add(cube)
  console.log('立方体已添加到场景')
  
  // 添加动画
  animate()
  console.log('动画已启动')
}

function animate() {
  requestAnimationFrame(animate)
  TWEEN.update()
  cube.rotation.x += 0.01
  cube.rotation.y += 0.01
  renderer.render(scene, camera)
}

async function enterVR() {
  if (!navigator.xr) {
    alert('您的浏览器不支持WebXR API')
    return
  }
  
  try {
    // 优先尝试immersive-vr，失败后尝试immersive-ar和inline
    const sessionTypes = ['immersive-vr', 'immersive-ar', 'inline']
    
    for (const type of sessionTypes) {
      try {
        const supported = await navigator.xr.isSessionSupported(type)
        if (supported) {
          const session = await navigator.xr.requestSession(type)
          renderer.xr.setSession(session)
          animate()
          return
        }
      } catch (error) {
        console.warn(`无法启动${type}会话:`, error)
      }
    }
    
    alert('您的设备不支持任何WebXR会话模式')
  } catch (error) {
    console.error('WebXR会话错误:', error)
    alert('无法启动WebXR会话: ' + error.message)
  }
}

onMounted(() => {
  initScene()
  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })
})
</script>

<style>
#xr-scene {
  width: 100%;
  height: 80vh;
}
</style>