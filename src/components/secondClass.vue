<template>
    <div>
    <canvas ref="scene_ref" class="scene"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue' 
import * as THREE from 'three'
import gsap from 'gsap'
const scene_ref = ref(null)
// 设置尺寸
const sizes = {
  width: 800,
  height: 600
}
onMounted(() => { 
  if (scene_ref.value) {
    const scene = new THREE.Scene()
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height)
    camera.position.z = 5
    camera.position.x = 0
    camera.position.y = 0
    // 在这个例子里面,我们要创建一个组,这个组就是可以同时进行操作的,例如我旋转30度,里面全部都会转动
    const group = new THREE.Group()
    // 将组添加到场景
    scene.add(group)
    // 如下是一种新的创建几何体的方法
    const cube1 = new THREE.Mesh(
      new THREE.BoxGeometry(1,1,1),
      new THREE.MeshBasicMaterial({ color: 'green' })
    )
    // 添加到组里面
    group.add(cube1)
    // 创建第二个几何体
    // const cube2 = new THREE.Mesh(
    //   new THREE.BoxGeometry(1,1,1),
    //   new THREE.MeshBasicMaterial({ color: 'red' })
    // )
    // cube2.position.x = 2
    // group.add(cube2)
    // 这个地方应该是能操控组, 果然可以 就是同时移动整个组里面所有内容
    // group.position.x = 1
    const axesHelper = new THREE.AxesHelper()
    scene.add(camera)
    scene.add(axesHelper)
    // 渲染
    const renderer = new THREE.WebGLRenderer({
      canvas: scene_ref.value
    })
    renderer.setSize(sizes.width, sizes.height)
    // 渲染
    // renderer.render(scene, camera)

    // 这个地方是创建一个记录时间的
    const clock = new THREE.Clock()
    // 动画
    // 通过gasp我们可以进行简单的移动操作,如下
    // 注意相当于我把tick里面操作移动的动作给提出来了而已,还是得重复调用动画函数的,否则看不到效果
    gsap.to(cube1.position, { duration:1, delay: 1, x: 2})
    gsap.to(cube1.position, { duration:1, delay: 2, x: 0})
    const tick = () => {
      // 每一次调用的时候,都会算出已经过去多久了
      // const elapsedTime = clock.getElapsedTime()
      // cube1.rotation.y += 0.01
      // cube1.position.y = Math.sin(elapsedTime)
      // cube1.position.x = Math.cos(elapsedTime)
      // camera.lookAt(cube1.position)
      renderer.render(scene, camera)
      // 进行操作
      window.requestAnimationFrame(tick)
    }
    tick()
  }
})
</script>

<style scoped>

</style>