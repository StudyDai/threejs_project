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
  import perper_svg from '/src/assets/01-GY.png'
  
  
  const scene_ref = ref(null)
  // 设置尺寸
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }
  onMounted(() => {
    // 初始化一个gui
    const gui = new lil.GUI()
    // 创建一个场景
    const scene = new THREE.Scene()
    // 创建纹理
    const textureLoader = new THREE.TextureLoader()
    const perper_img = textureLoader.load(perper_svg)
    // 创建粒子图形
    const count = 500
    const geometry = new THREE.BufferGeometry()
    const particlesArray = new Float32Array(count * 3)
    for (let index = 0; index < count * 3; index++) {
      // 创建粒子的位置
      particlesArray[index] = (Math.random() - 0.5) * 10    
    }
    geometry.setAttribute('position', new THREE.BufferAttribute(particlesArray, 3))
    // 下面的颜色跟材质身上的属性有关 如果想用这个color 那么添加属性
    // 这个属性会把你的顶点颜色和你的主颜色混合起来, 所以那如果只要看顶点颜色 那你就把材质里面的color去掉就行
    //  particle_material.vertexColors = true
    // geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))
    
    // 创建材质
    const particle_material = new THREE.PointsMaterial()
    particle_material.size = 0.1
    particle_material.sizeAttenuation = true 
    particle_material.transparent = true
    particle_material.alphaMap = perper_img
    particle_material.color = new THREE.Color('red')
    // 关闭材质的测试模式,会有透过去的特效 但是不推荐使用这个 因为这个是直接无视前面和后面的物体
    // 换句话说就是我里面插入一个立方体，他立方体后面的元素也会显示到前面来 这样会很诡异，在立方体后面
    // 不可见的地方就不可以看到才对
    // particle_material.depthTest = false
    // 深度写入 只要关掉 就不会影响到立方体的渲染和那些粒子的渲染，他还是会会按照深度图去画，只不过这次粒子可以穿透
    particle_material.depthWrite = false
    // 设置blend模式 就是让你的颜色进行叠加 混合 也就是说 后面一个例子 前面一个粒子 后面的例子会叠加前面的粒子颜色 导致变亮
    particle_material.blending = THREE.AdditiveBlending
    const mesh = new THREE.Points(geometry, particle_material)
    scene.add(mesh)
    // camera
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
    camera.position.x = 4
    camera.position.y = 2
    camera.position.z = 5
    scene.add(camera)
  
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
  
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    const clock = new THREE.Clock()
    function tick() {
      const elapsedTime = clock.getElapsedTime()
      // 我们要对粒子进行操作 这个做法的动画虽然可以实现 但是性能很一般 是不推荐的!!!
      for (let index = 0; index < count; index++) {
        const element = index * 3
        const x = geometry.attributes.position.array[element]
        geometry.attributes.position.array[element + 1] = Math.sin(elapsedTime + x * 0.9) 
      }
      // 修改了attributes之后记得让更新渲染
      // console.log(geometry)
      geometry.attributes.position.needsUpdate = true
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
  