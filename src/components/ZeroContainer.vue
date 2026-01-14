<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import {
  AmbientLight,
  DirectionalLight,
  PerspectiveCamera,
  Scene,
  SRGBColorSpace,
  Vector3,
  WebGLRenderer
} from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import modelUrl from '@/assets/zero.glb?url'

const containerRef = ref(null)

let renderer
let scene
let camera
let controls
let animationId

const setupScene = () => {
  const container = containerRef.value
  if (!container) return

  const { clientWidth, clientHeight } = container

  scene = new Scene()

  camera = new PerspectiveCamera(60, clientWidth / clientHeight, 0.1, 100)
  camera.position.set(0, 1.2, 3)

  const light = new DirectionalLight(0xffffff, 1)
  light.position.set(2, 4, 2)
  scene.add(light)
  scene.add(new AmbientLight(0xffffff, 0.5))

  renderer = new WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(clientWidth, clientHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.outputColorSpace = SRGBColorSpace
  container.appendChild(renderer.domElement)

  controls = new OrbitControls(camera, renderer.domElement)
  controls.target = new Vector3(0, 1, 0)
  controls.enableDamping = true

  const loader = new GLTFLoader()
  loader.load(
    modelUrl,
    (gltf) => {
      gltf.scene.position.set(0, 0, 0)
      gltf.scene.traverse((child) => {
        child.castShadow = true
        child.receiveShadow = true
      })
      scene.add(gltf.scene)
    },
    undefined,
    (error) => {
      console.error('Failed to load model:', error)
    }
  )

  window.addEventListener('resize', handleResize)

  animate()
}

const handleResize = () => {
  const container = containerRef.value
  if (!container || !renderer || !camera) return
  const { clientWidth, clientHeight } = container
  renderer.setSize(clientWidth, clientHeight)
  camera.aspect = clientWidth / clientHeight || 1
  camera.updateProjectionMatrix()
}

const animate = () => {
  animationId = requestAnimationFrame(animate)
  controls?.update()
  renderer?.render(scene, camera)
}

onMounted(() => {
  setupScene()
})

onBeforeUnmount(() => {
  cancelAnimationFrame(animationId)
  window.removeEventListener('resize', handleResize)
  controls?.dispose()
  renderer?.dispose()
  const container = containerRef.value
  if (container && renderer?.domElement) {
    container.removeChild(renderer.domElement)
  }
})
</script>

<template>
  <div ref="containerRef" class="zero-container"></div>
</template>

<style scoped>
.zero-container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background: white;
  overflow: hidden;
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.01);
}
</style>
