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
import * as lil from 'lil-gui'
const scene_ref = ref(null)
// 设置尺寸
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}
const cursor = {
  x: 0,
  y: 0
}
// 拿到原始距离旁边的距离
let canvas_left = 0, canvas_right = 0;
onMounted(() => { 
  if (scene_ref.value) {
    // 我们可以初始化一个DEBUG UI的面板
    const gui = new lil.GUI()
    const scene = new THREE.Scene()
    // 创建物体
    // const cube1 = new THREE.Mesh(
    //   new THREE.BoxGeometry(1,1,1,5,5,5),
    //   new THREE.MeshBasicMaterial({ color: 'red', wireframe: true })
    // )
    // scene.add(cube1)
    // 通过提供点坐标进行渲染图像,如下,一般3d的图像是三个值为一组x,y,z,而且得用浮点数32位的数组
    // const positionArray = new Float32Array([
    //   0,0,0,
    //   0,1,0,
    //   1,0,0
    // ])
    // 设置一个预属性 第二个参数是这组数据因多少个值为一组
    // const positionAttribute = new THREE.BufferAttribute(positionArray, 3)
    // 创建一个空的几何体,预设
    // const geometry = new THREE.BufferGeometry()
    // 应用属性
    // geometry.setAttribute('position', positionAttribute)
    const geometry = new THREE.BoxGeometry(1,1,1)
    // 设置属性
    const material = new THREE.MeshBasicMaterial({ color: 'yellow', wireframe: true })
    // 设置材质
    const mesh = new THREE.Mesh(geometry, material)
    scene.add(mesh)
    // 总共五个参数 属性 细分属性 最小值 最大值 步长
    gui.add(mesh.rotation, 'x',0, Math.PI*2, Math.PI / 6)
    // 上面的写法也可以写成链式调用 name是重命名的意思
    gui.add(mesh.position, 'x').min(-3).max(3).step(0.01).name('x位置')
    // 设置一个可不可视的按钮
    gui.add(mesh, 'visible').name("是否可见")
    // 控制线框
    gui.add(mesh.material, 'wireframe').name('开启线框')
    // 如果要更改物体的颜色的话,不能直接用gui.addEventListener,要用color有关的,如下
    // 我们可以写一个函数,丢到一个对象里面去,如下
    const paramsColor = {
      color: 0xff0000,
      spin() {
          // 旋转
          gsap.to(mesh.rotation, { duration: 1, y: mesh.rotation.y + Math.PI * 2 })
      }
    }
    // 当调色器发生改变的时候,重新设置材质的颜色,就是这么得到的,然后这个color对一个参数是要传递一个对象
    gui.addColor(paramsColor, 'color').onChange(() => {
        material.color.set(paramsColor.color)
    })
    gui.add(paramsColor, 'spin')
    // 创建一个辅助线
    // const axeshelper = new THREE.AxesHelper()
    // scene.add(axeshelper)
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
    camera.position.z = 5
    camera.position.x = 2
    scene.add(camera)
    const controls = new OrbitControls(camera, scene_ref.value)
    controls.enableDamping = true
    // 这个就是关闭控制的意思
    controls.enabled = true

    const renderer = new THREE.WebGLRenderer({
      canvas: scene_ref.value
    })
    renderer.setSize(sizes.width, sizes.height)
    // 在这个地方设置他的像素比,可以导入我们电脑的像素比给threejs
    // 这个分辨率一般在1-2就够用了,所以用min来限制一下
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    const tick = () => {
      controls.update()
      renderer.render(scene, camera)
      window.requestAnimationFrame(tick)
    }
    tick()
    // 监听重新拉动页面 要写在第一次执行之后!! 切记
    window.addEventListener('resize', () => {
      sizes.width = window.innerWidth
      sizes.height = window.innerHeight
      // 更新摄像机的比例
      camera.aspect = sizes.width / sizes.height
      // 通知three 重新渲染摄像机
      camera.updateProjectionMatrix()
      // 重新设置渲染器大小
      renderer.setSize(sizes.width, sizes.height)
    })
    // 监听双击 我们要实现双击全屏的效果
    window.addEventListener('dblclick', () => {
      // 针对苹果的浏览器我们要设置如下
      const fullscreenElement = document.fullscreenElement || document.webkitFullscreenElement
      // 这个地方要判断是否是苹果的 否则就要用
      if (!fullscreenElement) {
        // 如果不存在的话 就进来判断下 是否能用打开全屏的操作
        if (scene_ref.value.requestFullscreen) {
          // 那么就调用
          scene_ref.value.requestFullscreen()
        }
        else if (scene_ref.value.webkitRequestFullscreen) {
          scene_ref.value.webkitRequestFullscreen()
        }
      }
      else {
        if (document.exitFullscreen) {
          // 如果关闭函数存在就调用关闭函数
          document.exitFullscreen()
        } 
        else if (document.webkitExitFullscreen) {
          // 如果苹果的关闭函数在的话 就用这个
          document.webkitExitFullscreen()
        }
      }
    })
  }
})
</script>

<style scoped>
    * {
    margin: 0;
    padding: 0;
  }
</style>