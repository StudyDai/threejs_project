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
  import demo_img from '/src/assets/demo.png'
  import real_global_01 from '/src/assets/real_global_03.png'
  import real_global_02 from '/src/assets/real_global_02.png'
  // 导入字体加载器 这个是133版本之后的改动
  import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js'
  import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js'
  // 引入字体数据
  import fontJson from '@/assets/fonts/helvetiker_regular.typeface.json'
  // 矩形灯的helper要导入
  import { RectAreaLightHelper } from 'three/examples/jsm/helpers/RectAreaLightHelper.js'
  // 导入字体加载器可以加载多个字体
  const fontLoader = new FontLoader()
  const scene_ref = ref(null)
  // 设置尺寸
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }
  // 创建一个纹理加载器
  onMounted(() => {
    // 初始化一个gui
    const GUI = new lil.GUI()
    // 创建一个场景
    const scene = new THREE.Scene()
    const AxesHelper = new THREE.AxesHelper()
    scene.add(AxesHelper)
    // 创建一个形状
    const geometry = new THREE.BoxGeometry(1,1,1)
    // 创建一个标准网格 这个材质是需要光源的 否则是看不清晰物体的
    const material = new THREE.MeshStandardMaterial()
    // 增加粗糙度
    material.roughness = 0.4
    // 创建环境光 这种是光从四面八方打到物体身上去的 一般强度就是0-1,不会更大了
    // 环境光会均匀的照射到整个物体的各个面 所以多数让他充当反射或者是阴影的组成部分
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
    // ambientLight.color = new THREE.Color('0xffffff') 设置光颜色 等同于上面一步
    // ambientLight.intensity = 0.5 这个是设置光强度
    // scene.add(ambientLight)
  
    // 方向光(定向光) 照射出来的光都是平行的 默认这个光是从正上方打向空间中心的
    const directionLight = new THREE.DirectionalLight(0x00fffc, 0.2)
    // 例如把光改到右边去,那则是从右边打到空间中心去的
    directionLight.position.set(1, 0.25, 0)
    // scene.add(directionLight)
    const directionalLightHelper = new THREE.DirectionalLightHelper(directionLight, 0.2)
    // scene.add(directionalLightHelper)
  
    // 半球形光 就是一个颜色占一半 三个参数 渐变效果 俩颜色 分别是红色和蓝 第三个参数是强度
    const hemisphereLight = new THREE.HemisphereLight(0xff0000, 0x0000ff, 0.3) 
    // scene.add(hemisphereLight)
    // 设置一个半球形光的辅助助手 传递俩参数 分别是光和辅助助手大小
    const HemisphereLightHelper = new THREE.HemisphereLightHelper(hemisphereLight, 0.2)
    // scene.add(HemisphereLightHelper)
    // 点光源
    const pointLight = new THREE.PointLight(0xFF9000, 1.5, 10, 2)
    const pointLightHelper = new THREE.PointLightHelper(directionLight, 0.2)
    // 可能是默认是在0,0,0,所以光又很暗,所以看不到光点
    // GUI.add(pointLight.position, 'z').min(0).max(1).name('光源距离')
    // GUI.add(pointLight.position, 'x').min(0).max(1).name('光源x距离')
    // GUI.add(pointLight.position, 'y').min(0).max(1).name('光源y距离')
    pointLight.position.set(1, 1, 1)
    // scene.add(pointLight, pointLightHelper)
  
    // 矩形区域光 第三和第四个参数定义的是灯光的宽高 第二个参数是定义灯光的亮度
    // 只支持标准网格材质和物理材质
    const rectAreaLight = new THREE.RectAreaLight(0x4e00ff, 2, 1, 1)
    rectAreaLight.position.set(-1.5 ,0, 1.5)
    // 先移动再调节方向 默认vector3就是0,0,0 看向圆心
    rectAreaLight.lookAt(new THREE.Vector3())
    // 帮助者
    const rectAreaLightHelper = new RectAreaLightHelper(rectAreaLight)
    scene.add(rectAreaLight, rectAreaLightHelper)
  
    // 聚光灯
    // 颜色 强度 光距(能打多远) 光源打开的角度 高斯模糊边缘 这个不知道是啥 一般保持1
    const spotLight = new THREE.SpotLight(0x78ff00, 1, 4, Math.PI * 0.15, 0.25, 1)
    spotLight.position.set(0,1,1)
    // 如果要调整聚光灯的照射方向得将目标加进去画面
    spotLight.target.position.x = -0.75
    // scene.add(spotLight.target) 聚光灯是没有尺寸大小的 所以第二个值不用设置
    const spotLightHelper = new THREE.SpotLightHelper(spotLight)
    // scene.add(spotLight, spotLightHelper)
    // 应用下
    const mesh = new THREE.Mesh(geometry, material)
    scene.add(mesh)
    // 创建一个平面
    const plane = new THREE.Mesh(
      new THREE.PlaneGeometry(5, 5),
      material
    )
    plane.rotation.x = -Math.PI * 0.5
    plane.position.y = -0.65
    GUI.add(plane.rotation, 'y').min(0).max(Math.PI * 2).step(0.02)
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height)
    camera.position.x = 1
    camera.position.y = 1
    camera.position.z = 2
    // 摄像机添加到场景中
    scene.add(camera, plane)
    // 让环境可以拖拽
    const controls = new OrbitControls(camera, scene_ref.value)
    // 开阻尼
    controls.enableDamping = true
    // 创建渲染器
    const renderer = new THREE.WebGLRenderer({
      canvas: scene_ref.value
    })
    // 设置渲染器大小
    renderer.setSize(sizes.width, sizes.height)
    function tick() {
      controls.update()
      // 渲染器渲染场景+摄像机
      renderer.render(scene, camera)
      requestAnimationFrame(tick)
    }
    tick()
  })
  </script>
  
  <style>
    * {
      margin: 0;
      padding: 0;
    }
  #app {
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: gray;
  }
  </style>
  