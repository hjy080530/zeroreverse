<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import {
  AmbientLight,
  Box3,
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
  // 렌더러를 붙일 DOM 컨테이너가 준비됐는지 확인
  const container = containerRef.value
  if (!container) return

  const { clientWidth, clientHeight } = container

  // 씬과 카메라 기본 세팅
  scene = new Scene()

  camera = new PerspectiveCamera(60, clientWidth / clientHeight, 0.1, 100)
  camera.position.set(0, 1.2, 3)

  // 기본 조명: 방향광 + 주변광
  const light = new DirectionalLight(0xffffff, 1)
  light.position.set(2, 4, 2)
  scene.add(light)
  scene.add(new AmbientLight(0xffffff, 0.5))

  // 안티앨리어싱 포함한 렌더러 생성 후 컨테이너에 부착
  renderer = new WebGLRenderer({ antialias: true, alpha: true })
  renderer.setSize(clientWidth, clientHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.outputColorSpace = SRGBColorSpace
  container.appendChild(renderer.domElement)

  // 마우스 드래그로 회전할 수 있는 오빗 컨트롤
  controls = new OrbitControls(camera, renderer.domElement)
  controls.target = new Vector3(0, 1, 0)
  controls.enableDamping = true

  // GLB 로드 시작
  const loader = new GLTFLoader()
  loader.load(modelUrl, handleModelLoad, undefined, (error) => {
    console.error('Failed to load model:', error)
  })

  window.addEventListener('resize', handleResize)

  animate()
}

const handleResize = () => {
  // 컨테이너 크기에 맞춰 렌더러/카메라 비율 갱신
  const container = containerRef.value
  if (!container || !renderer || !camera) return
  const { clientWidth, clientHeight } = container
  renderer.setSize(clientWidth, clientHeight)
  camera.aspect = clientWidth / clientHeight || 1
  camera.updateProjectionMatrix()
}

const animate = () => {
  // 프레임마다 컨트롤 업데이트 후 렌더
  animationId = requestAnimationFrame(animate)
  controls?.update()
  renderer?.render(scene, camera)
}

const handleModelLoad = (gltf) => {
  // 로드된 모델을 중심 기준으로 재배치 후 카메라 프레이밍
  const root = gltf.scene
  const box = new Box3().setFromObject(root)
  const center = new Vector3()
  box.getCenter(center)

  root.position.sub(center)

  const size = new Vector3()
  box.getSize(size)
  const maxDim = Math.max(size.x, size.y, size.z)
  const fov = (camera.fov * Math.PI) / 180
  const distance = maxDim / (2 * Math.tan(fov / 2))
  camera.position.set(0, size.y * 0.5, distance * 1.6)
  controls.target.set(0, size.y * 0.25, 0)
  controls.update()

  // 그림자 처리 (필요 시 재사용)
  root.traverse((child) => {
    child.castShadow = true
    child.receiveShadow = true
  })

  scene.add(root)
}

onMounted(() => {
  // 컴포넌트가 마운트된 뒤 씬을 초기화
  setupScene()
})

onBeforeUnmount(() => {
  // 애니메이션/이벤트/리소스 정리
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
