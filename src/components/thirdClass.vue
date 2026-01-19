<template>
    <div>
    <canvas ref="scene_ref" class="scene"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue' 
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import gsap from 'gsap'
const scene_ref = ref(null)
// 设置尺寸
const sizes = {
  width: 800,
  height: 600
}
const cursor = {
  x: 0,
  y: 0
}
// 拿到原始距离旁边的距离
let canvas_left = 0, canvas_right = 0;
// window.addEventListener('mousemove', (event) => {
//   cursor.x = (event.clientX - canvas_left) / sizes.width - 0.5
//   cursor.y = (event.clientY - canvas_right) / sizes.height - 0.5
// })
onMounted(() => { 
  if (scene_ref.value) {
    let canvas_details = scene_ref.value.getBoundingClientRect()
    canvas_left = canvas_details.left
    canvas_right = canvas_details.top
    console.log(canvas_left, canvas_right)
    // 摄像机分为俩种常用的 之前用到的是视觉摄像机
    // 今天介绍的是自动照相机 提供四个参数 分别代表四个点 他是一个矩形的拍摄范围,而不像之前的
    // THREE.OrthographicCamera(左,右,上,下)
    // 今天介绍的拍摄范围如下
    //  ↑ → → → ↓
    //  ↑       ↓
    //  ↑       ↓
    //  ↑ ← ← ← ↓
    // 之前的是做一个上下闭合的拍摄范围也就是 ↑↓
    // 顺序是 左右 上下
    // const camera = new THREE.OrthographicCamera()
    // 一般使用的方法就是 -1*宽高比 1*宽高比 -1 1 0.1 100
    // 如果不乘宽高比的话,你的形状就会被压缩,例如你的视角只有1:1,但是你要去看一个3:4的,那么你的产品就会被拉伸,相当于你的产品在1:1的环境里面去被迫去放到3:4里面
    // 我们来玩相机了 鼠标往哪边走 相机往哪边走
    // 创建一个屏幕
    const scene = new THREE.Scene()
    // 创建物体
    const cube1 = new THREE.Mesh(
      new THREE.BoxGeometry(1,1,1),
      new THREE.MeshBasicMaterial({ color: 'red' })
    )
    scene.add(cube1)
    // 创建一个辅助线
    const axeshelper = new THREE.AxesHelper()
    scene.add(axeshelper)
    // 创建照相机 0.1是最小可视范围 100是最大可视范围 在最大可视以外的会被遮住
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
    camera.position.z = 5
    camera.position.x = 2
    scene.add(camera)
    // 创建控制器
    const controls = new OrbitControls(camera, scene_ref.value)
    // 可以添加阻尼,让他有惯性 切记,一旦开了这个就得让这个update函数在tick函数里头,就是一直去更新
    controls.enableDamping = true
    // 可以设置控制器默认盯着的位置,通过target.x/y/z来控制,默认是0,0,0,我们可以设置他一直盯着0,2,0这个点去进行操作
    // controls.target.y = 5
    // 这个更新的函数必须调用,否则上面的target设置无效
    // controls.update()
    const renderer = new THREE.WebGLRenderer({
      canvas: scene_ref.value
    })
    renderer.setSize(sizes.width, sizes.height)
    const tick = () => {
      // 这个写法只能看到三个面,看不到最后那个面 这个写法就是让我们可以全面看到方快
      // camera.position.x = cursor.x * 3
      // camera.position.y = -cursor.y * 3
      // camera.position.x = Math.sin(cursor.x * Math.PI * 2) * 3
      // camera.position.z = Math.cos(cursor.x * Math.PI * 2) * 3
      // camera.position.y = -cursor.y * 4
      // 设置摄像机看向的中心 默认是看向原点(0,0,0) 切记是设置摄像机的位置 而不是物体的!! 妈的这是个坑
      // Vector3() == (0,0,0)
      // camera.lookAt(new THREE.Vector3())
      controls.update()
      renderer.render(scene, camera)
      window.requestAnimationFrame(tick)
    }
    tick()
  }
})
</script>

<style scoped>
#app {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: gray;
}
</style>