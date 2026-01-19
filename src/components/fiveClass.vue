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
  // 如下引入图片,再到应用其实是原理来的,他有一个纹理渲染器,可以代替这些步骤
  // 创建一个图片
  // const image = new Image()
  // 创建纹理 默认image是空的,所以是没文理的
  // let texture = new THREE.Texture(image)
  // image.onload = (resp) => {
    // 直接调用纹理身上的函数,告诉他需要去重新拿去image了
  //   texture.needsUpdate = true
  // }
  // image.src = demo_img
  // 创建一个纹理加载管理器
  const loadingManager = new THREE.LoadingManager()
  // 设置监听 他有很多个函数 onstart onloaded onprogress onerror 多个 都是驼峰命名法 我就不写大写了
  loadingManager.onStart = () => {}
  //  接下来使用我们的纹理加载器
  const textureLoader = new THREE.TextureLoader(loadingManager)
  const texture = textureLoader.load(
    demo_img,
    () => {
      console.log('加载函数')
    },
    () => {
      conosle.log('进度条函数,基本不用')
    },
    () => {
      console.log('报错函数')
    }
  )
  // 这个地方我们引入最大滤镜和最小滤镜,这两个滤镜的默认都是线性过滤
  // 用于控制纹理在放大 / 缩小时的显示效果，核心作用是定义纹理的像素插值规则
  // 但是,当我们导入的图片太小张的时候,我们就要将放大滤镜设置为临近过滤,这样子你的图片会被锐化,变清晰,分辨率也会变高
  // Mipmap（多级渐远纹理）：是一组预先生成的、分辨率依次降低的纹理副本（
  // 比如原纹理是 1024x1024，mipmap 会包含 512x512、256x256 等尺寸），
  // 用于优化纹理在缩小显示时的性能和效果。
  // 如果我们下面使用的nearestFilter的话 那么我们可以吧这个mipmaps禁用掉来提高性能
  texture.generateMipmaps = false
  // 控制纹理被放大显示（比如纹理实际是 256x256，但要显示成 512x512）时的像素插值方式。
  // THREE.NearestFilter（最近邻过滤）：直接取距离当前像素最近的纹理像素颜色，效果是纹理放大后呈现 “像素化” 风格（边缘锐利、无模糊），性能开销低。
  // THREE.NearestFilter	像素化、无模糊、性能高	像素风格、低性能设备
  // THREE.LinearFilter	平滑插值、模糊过渡、性能中等	追求纹理细腻的场景
  texture.magFilter = THREE.NearestFilter
  texture.minFilter = THREE.NearestFilter
  // 可以设置他x轴y轴分别复制多少份,例如x两张 y三张
  // texture.repeat.x = 2
  // texture.repeat.y = 2
  // 设置排列的模式 RepeatWrapping 就是正常按顺序去排
  // wrapS是水平 wrapT是垂直
  // texture.wrapS = THREE.RepeatWrapping
  // texture.wrapT = THREE.RepeatWrapping
  // MirroredRepeatWrapping是镜像去拍
  // texture.wrapT = THREE.MirroredRepeatWrapping
  // texture.wrapS = THREE.MirroredRepeatWrapping
  
  // 这个是偏移
  // texture.offset.x = 0.5
  // texture.offset.y = 0.5
  
  // 我们可以通过设置中心点,来让图片旋转不一样 默认是左下角 我们可以定位到中间
  // texture.center.x = 0.5
  // texture.center.y = 0.5
  // 这个是旋转 这个是旋转45度 向左边转动
  // texture.rotation = Math.PI / 4
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
  onMounted(() => {
    // 创建一个场景
    const scene = new THREE.Scene()
    // 创建一个形状
    const geometry = new THREE.BoxGeometry(1,1,1)
    // 添加材质
    // const material = new THREE.MeshBasicMaterial({ color: 'red' })
    // 添加纹理
    const material = new THREE.MeshBasicMaterial({ map: texture })
    // 生成材质作用的物体
    const mesh = new THREE.Mesh(geometry, material)
    // 物体添加到场景中
    scene.add(mesh)
    // 创建一个摄像机
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height)
    camera.position.z = 5
    // 摄像机添加到场景中
    scene.add(camera)
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
  