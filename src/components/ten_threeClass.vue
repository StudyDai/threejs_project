<template>
    <div>
        <canvas ref="scene_ref" class="scene"></canvas>
          <h1>this is the one text</h1>
          <h1>this is the two text</h1>
          <h1>this is the three text</h1>
      </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue'
  import * as lil from 'lil-gui'
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
  // 引入字体数据
  import fontJson from '@/assets/fonts/helvetiker_regular.typeface.json'
  import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js'
  import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js'
  import gradient_img from '/src/assets/20/3.jpg'
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
  // 监听滚动
  let screenY_num = 0
  window.onscroll = () => {
    screenY_num = window.scrollY
  }
  // 监听鼠标移动
  window.addEventListener('mousemove', (event) =>
  {
      cursor.x = event.clientX / sizes.width - 0.5
      cursor.y = event.clientY / sizes.height - 0.5
  })
  // 创建一个纹理
  const textloader = new THREE.TextureLoader()
  // 创建纹理
  const gradientLoader = textloader.load(gradient_img)
  // 要改变渲染的模式,不要弄这个很模糊的滤镜
  gradientLoader.magFilter = THREE.NearestFilter
  onMounted(() => {
    if (scene_ref.value) {
      // Material
      const material = new THREE.MeshToonMaterial({
          color: guiObj.color,
          gradientMap: gradientLoader
      })
      gui.addColor(guiObj,'color').onChange(() => {
        console.log('激活')
            material.color.set(guiObj.color)
      })
      // Objects
      const mesh1 = new THREE.Mesh(
          new THREE.TorusGeometry(1, 0.4, 16, 60),
          material
      )
      const mesh2 = new THREE.Mesh(
          new THREE.ConeGeometry(1, 2, 32),
          material
      )
      const mesh3 = new THREE.Mesh(
          new THREE.TorusKnotGeometry(0.8, 0.35, 100, 16),
          material
      )
  
      mesh1.position.y = - instance * 0
      mesh2.position.y = - instance * 1
      mesh3.position.y = - instance * 2
  
      mesh1.position.x = 2
      mesh2.position.x = -4
      mesh3.position.x = 2
  
      scene.add(mesh1, mesh2, mesh3)
      const meshList = [mesh1, mesh2, mesh3]
  
      // 添加一个平行光
      const directionLight = new THREE.DirectionalLight()
      // 设置光的位置
      directionLight.position.set(1,1,0)
      // 加入
      scene.add(directionLight)
      // 创建摄像机
      const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
      // 设置摄像机的位置
      camera.position.set(0,0,4)
      // 摄像机添加
      scene.add(camera)
  
      // 加一个原点坐标系
      const axesHelper = new THREE.AxesHelper()
      scene.add(axesHelper)
      // 加个文字进来
      // 创建字体加载器
      const fontLoader = new FontLoader()
      // 纹理
      const font = fontLoader.parse(fontJson)
      const textGeometry = new TextGeometry('DEMO TEST', {
        font,
        size: 0.5,
        depth: 0.2,
        curveSegments: 12, // 调节这个可以让三角形不是很多 提高性能 大概6左右 但是就没那么细节
        bevelEnabled: true,
        bevelThickness: 0.03, // 这个是x和y轴的斜角度 当你在调节中心位置的时候 要减掉它
        bevelSize: 0.02, // 这个是z轴的斜角度
        bevelOffset: 0,
        bevelSegments: 5 
      })
      // 一样的材质
      const text = new THREE.Mesh(textGeometry, material)
      textGeometry.computeBoundingBox()
      textGeometry.translate(
        -(textGeometry.boundingBox.max.x - 0.02) / 2,
        -(textGeometry.boundingBox.max.y - 0.02) / 2,
        -(textGeometry.boundingBox.max.z - 0.03) / 2
      )
      // scene.add(text)
      // 设置渲染器
      const renderer = new THREE.WebGLRenderer({
        canvas: scene_ref.value,
        alpha: true, // 你的渲染器背景就会变透明
      })
      // 设置渲染器大小
      renderer.setSize(sizes.width, sizes.height)
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
      // 拿到执行的时间
      const clock = new THREE.Clock()
      // 直接渲染
      const cameraGroup = new THREE.Group()
      cameraGroup.add(camera)
      scene.add(cameraGroup)
      const tick = () => {
        //  移动满了就是一整个屏幕 那我是不是第一个默认就是0
        //  当移动到第二个出现的时候,是不是已经经过一个满屏了去乘 这个地方用组
        cameraGroup.position.y = - screenY_num / sizes.height * instance
        camera.position.x = cursor.x
        camera.position.y = - cursor.y
        // 时间
        const elapsedTime = clock.getElapsedTime()
        meshList.forEach(mesh => {
          mesh.rotation.x = elapsedTime * 0.2
          mesh.rotation.y = elapsedTime * 0.3
        })
        renderer.render(scene, camera)
        requestAnimationFrame(tick)
      }
      tick()
    }
  
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
    .text_container {
      position: absolute;
      top: 0;
      left: 0;
    }
    h1 {
      display: flex;
      align-items: center;
      height: 100vh;
      position: relative;
      font-family: 'Cabin', sans-serif;
      color: #ffeded;
      text-transform: uppercase;
      font-size: 7vmin;
      padding-left: 10%;
      padding-right: 10%;
    }
    h1:nth-of-type(2n) {
      justify-content: flex-end;
      padding-right: 100px;
    }
    #app {
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: transparent;
    }
  </style>