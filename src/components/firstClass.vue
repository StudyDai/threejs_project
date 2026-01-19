<template>
    <div>
    <canvas ref="scene_ref" class="scene"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue' 
import * as THREE from 'three'
const scene_ref = ref(null)
// 设置尺寸
const sizes = {
  width: 800,
  height: 600
}
onMounted(() => {
if (scene_ref.value) {
  // 创建场景
  const scene = new THREE.Scene()
  // 创建Object
  const geometry = new THREE.BoxGeometry(1,1,1)
  // 材质
  // wireframe: true 这个参数是用来显示线条的,小正方是背面的内容
  const material = new THREE.MeshBasicMaterial({ color: 'green' })
  // 应用材质的对象
  const mesh = new THREE.Mesh(geometry, material)
  // 这地方可以对对象进行缩放
  mesh.scale.x = 2
  mesh.scale.y = 0.5
  // 默认这样子去书写xy旋转的时候他会变得特别的怪
  // 如下这个reorder的含义就是让你的xyz的轴跟着物体动 会按照你设置的顺序来执行
  // mesh.rotation.reorder('YXZ')
  // 默认你的坐标是不会动的,也就是你的xyz是在原地不动的
  // 不管你的物体咋动 你的坐标就是固定在那里,二这个reorder会跟跟着动
  // 旋转 我知道了 这个渲染是按照角度来的 一圈是2π,半圈是π
  mesh.rotation.y = Math.PI / 4
  mesh.rotation.x = Math.PI / 4
  // 将物品丢到场景里面
  scene.add(mesh)

  // 创建摄影机
  const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height)
  // 将照相机往外走,也就是说我们的物品会往前走
  camera.position.z = 3
  camera.position.x = 1
  camera.position.y = 1
  // 这个是x,y,z的复合写法
  // camera.position.set(x,y,z)
  // geometry的normalize是让你的物体和你的摄像机的距离保持为1,好像还挺有用的,后面再看
  // 加入到场景里面
  scene.add(camera)

  // 添加一个轴辅助线 可以传参 x,y,z三个轴的长度,是默认长度的倍数
  const axesHelper = new THREE.AxesHelper()
  // 添加进去场景
  scene.add(axesHelper)

  // 创建渲染器
  const renderer = new THREE.WebGLRenderer({
    canvas: scene_ref.value
  })
  // 设置渲染器的大小
  renderer.setSize(sizes.width, sizes.height)

  // 渲染
  renderer.render(scene, camera)
}
})
</script>

<style scoped>

</style>
  