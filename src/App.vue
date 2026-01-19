<template>
  <div>
      <canvas ref="scene_ref" class="scene"></canvas>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import * as lil from 'lil-gui'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
// 导入物理库
import cannon from 'cannon'
// 创建一个物理世界 必须做的异步
const world = new cannon.World()
// 添加重力
world.gravity.set(0, -9.82, 0)
// 创建一个半径一样的球体 先创建形状,跟我们three创建mesh一样 先创建geometry
const sphereShape = new cannon.Sphere(0.5)
const sphereBody = new cannon.Body({
  mass: 1, //这个是质量,质量越大,越能反弹其他物体
  position: new cannon.Vec3(0, 3, 0), // 定义位置 有点像three的vector3
  shape: sphereShape
})
world.addBody(sphereBody)

// 创建一个平面
const plane = new cannon.Plane()
const planeBody = new cannon.Body()
// 开始填充
planeBody.mass = 0 // 这个0的意思就是告诉threejs我这个物体是保持不动的,你可以往我上面丢东西
// 设置平面的正确位置 注意 平面一定要正面朝上,否则你的物体会消失,就好比你的
// 地面是放在地球上的,你的地面下面就是地球以外的世界,所以是看不到的
// 物理空间里面进行旋转比较复杂,需要选择旋转点和旋转角度,如下
planeBody.quaternion.setFromAxisAngle(
  new cannon.Vec3(-1, 0, 0),
  Math.PI * 0.5
)
planeBody.addShape(plane)
world.addBody(planeBody)

const scene_ref = ref(null)
  // Scene
const scene = new THREE.Scene()

const gui = new lil.GUI()

// 创建一个对象用来存储gui控制的内容
const guiObj = {
  color: '#0xffffff'
}
const cursor = {
  x: 0,
  y: 0
}
// 移动的距离
const instance = 6
// 设置尺寸
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}
window.addEventListener('resize', () =>
{
    // Update sizes
    sizes.width = window.innerWidth
    sizes.height = window.innerHeight

    // Update camera
    camera.aspect = sizes.width / sizes.height
    camera.updateProjectionMatrix()

    // Update renderer
    renderer.setSize(sizes.width, sizes.height)
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
})
onMounted(() => {
/**
 * Camera
 */
// Base camera
const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
camera.position.set(- 3, 3, 3)
scene.add(camera)

// Controls
const controls = new OrbitControls(camera, scene_ref.value)
controls.enableDamping = true

/**
 * Test sphere
 */
 const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(0.5, 32, 32),
    new THREE.MeshStandardMaterial({
        metalness: 0.3,
        roughness: 0.4,
        color: 'green'
    })
)
sphere.castShadow = true
sphere.position.y = 0.5
scene.add(sphere)
/**
 * Floor
 */
 const floor = new THREE.Mesh(
    new THREE.PlaneGeometry(10, 10),
    new THREE.MeshStandardMaterial({
        color: '#777777',
        metalness: 0.3,
        roughness: 0.4,
        envMapIntensity: 0.5
    })
)
floor.receiveShadow = true
floor.rotation.x = - Math.PI * 0.5
scene.add(floor)
/**
 * Lights
 */
const ambientLight = new THREE.AmbientLight(0xffffff, 2.1)
scene.add(ambientLight)

const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6)
directionalLight.castShadow = true
directionalLight.shadow.mapSize.set(1024, 1024)
directionalLight.shadow.camera.far = 15
directionalLight.shadow.camera.left = - 7
directionalLight.shadow.camera.top = 7
directionalLight.shadow.camera.right = 7
directionalLight.shadow.camera.bottom = - 7
directionalLight.position.set(5, 5, 5)
scene.add(directionalLight)
/**
 * Renderer
 */
const renderer = new THREE.WebGLRenderer({
    canvas: scene_ref.value
})
renderer.shadowMap.enabled = true
renderer.shadowMap.type = THREE.PCFSoftShadowMap
renderer.setSize(sizes.width, sizes.height)
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
const clock = new THREE.Clock()
let oldElapsedTime = 0
const tick = () =>
{
    const elapsedTime = clock.getElapsedTime()
    const deltaTime = elapsedTime - oldElapsedTime
    oldElapsedTime = elapsedTime
    // 更新我们的物理世界 第一个参数(基础时间,算出每一毫秒的执行时间) 其实可以写死1/60无所谓
    // 你电脑多少帧就写/多少,第三个数固定写3就行
    // 第二个参数就是算你距离上次渲染过去了多少秒,其实就是用elapsed那个时间
    world.step(1 / 60, deltaTime, 3)
    // sphere.position.x = sphereBody.position.x
    // sphere.position.y = sphereBody.position.y
    // sphere.position.z = sphereBody.position.z
    // 等同于上面三行
    sphere.position.copy(sphereBody.position)
    // Update controls
    controls.update()

    // Render
    renderer.render(scene, camera)

    // Call tick again on the next frame
    window.requestAnimationFrame(tick)
}

tick()
})

</script>

<style>
  .scene {
    position: fixed;
    top: 0;
    left: 0;
  }
* {
    margin: 0;
    padding: 0;
  }
  body {
    width: 100vw;
  }
  html {
    background-color: #000;
  }
  #app {
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: transparent;
  }
</style>