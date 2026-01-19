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
  const scene_ref = ref(null)
    // Scene
  const scene = new THREE.Scene()
  
  
  const paramMeters = {}
  paramMeters.count = 10000
  paramMeters.size = 0.02
  paramMeters.radius = 6
  paramMeters.branchNum = 5
  paramMeters.spin = 0.36
  paramMeters.randomness = 0.2
  paramMeters.randomnessPower = 3
  // 创建颜色
  paramMeters.insideColor = '#ff6030'
  paramMeters.ousideColor = '#1b3984'
  
  /**
   * 初始化函数
   */
  let particle_array = null,
  particle_geometry = null,
  particle_point_material = null,
  points = null,
  particle_colors = null
  const generateGalaxy = () => {
    // 这里有个一定要做的事,就是我每次操作gui的时候 他是不断重复这个动作,就是一直加,没有减掉的趋势
    // 所以这个地方要先释放内存,然后清理场景里面的点,再操作如下
    if (particle_array) {
      // 有东西 来了 开始释放
      particle_geometry.dispose()
      particle_point_material.dispose()
      scene.remove(points)
    }
    // 生成500个随机丢的粒子 创建坐标
    particle_array = new Float32Array(paramMeters.count * 3)
    // 创建颜色
    particle_colors = new Float32Array(paramMeters.count * 3)
    // 创建两个颜色
    const colorinSide = new THREE.Color(paramMeters.insideColor)
    const coloroutSide = new THREE.Color(paramMeters.ousideColor)
    // 有一个插值运算 就是让你从一个颜色过渡到另一个颜色 第二个参数就是如何过渡 0.5就是模糊过渡 1就是层次分明
    // 但是最好不要直接使用这个值来进行操作,我们于复制一个
    // colorinSide.lerp(coloroutSide, 0.5)
    // 创建几何体
    particle_geometry = new THREE.BufferGeometry()
    for (let index = 0; index < paramMeters.count * 3; index++) {
      const i3 = index * 3
      // 做银河 先吧x都定位到一条线上面
      const radius = Math.random() * paramMeters.radius
      // 创建一个新的旋转角度
      const spinAngle = radius * paramMeters.spin
      // 创建随机数
      // const randomX = (Math.random() - 0.5) * paramMeters.randomness
      // const randomY = (Math.random() - 0.5) * paramMeters.randomness
      // const randomZ = (Math.random() - 0.5) * paramMeters.randomness
      // 这个随机性更强
      const randomX = Math.pow(Math.random(), paramMeters.randomnessPower) * (Math.random() < 0.5 ? 1  : -1)
      const randomY = Math.pow(Math.random(), paramMeters.randomnessPower) * (Math.random() < 0.5 ? 1  : -1)
      const randomZ = Math.pow(Math.random(), paramMeters.randomnessPower) * (Math.random() < 0.5 ? 1  : -1)
      // 得到角度 
      const branchAngle = (index % paramMeters.branchNum) / paramMeters.branchNum * Math.PI * 2
      particle_array[i3] = Math.cos(branchAngle + spinAngle) * radius + randomX
      particle_array[i3 + 1] = randomY
      particle_array[i3 + 2] = Math.sin(branchAngle + spinAngle) * radius + randomZ
  
  
      // 在这个地方使用插值 threejs里面有很多类都有克隆的属性方法
      const mixedColor = colorinSide.clone()
      // 这个地方如果直接写0.5 他会混入,所以这个地方我们判断一下,根据半径 从0到1的区间内
      mixedColor.lerp(coloroutSide, radius / paramMeters.radius)
  
      // 直接使用插值之后的rgb
      particle_colors[i3] = mixedColor.r
      particle_colors[i3 + 1] = mixedColor.g
      particle_colors[i3 + 2] = mixedColor.b
  
  
      // 颜色
      // particle_colors[i3] = 1
      // particle_colors[i3 + 1] = 0
      // particle_colors[i3 + 2] = 0
    }
    // 将坐标设置给元素 设置他的值是一个数组 并且要是32位的float 并且是设置第二个参数是每多少位是一组
    particle_geometry.setAttribute('position', new THREE.BufferAttribute(particle_array, 3))
    // 将颜色加进去
    particle_geometry.setAttribute('color', new THREE.BufferAttribute(particle_colors, 3))
    // 创建材质
    particle_point_material = new THREE.PointsMaterial({
      size: paramMeters.size,
      sizeAttenuation: true,  // 开启粒子削弱, 越远越暗
      depthWrite: false,  // 关闭深度检测
      blending: THREE.AdditiveBlending,  // 开启混合模式
      vertexColors: true, // 我们要使用顶点颜色
    })
  
    // 创建点
    points = new THREE.Points(particle_geometry, particle_point_material)
    // 添加
    scene.add(points)
  }
  
  generateGalaxy()
  
  // 创建gui
  const gui = new lil.GUI()
  // onFinishChange 这个函数只会等到你松开手才执行 就跟input的input一样,嘎嘎执行 但是change只有你失去焦点的时候才会触发
  gui.add(paramMeters, 'count').min(100).max(1000).step(10).onFinishChange(generateGalaxy)
  gui.add(paramMeters, 'branchNum').min(1).max(10).step(1).onFinishChange(generateGalaxy)
  gui.add(paramMeters, 'size').min(0.001).max(0.1).step(0.001).onFinishChange(generateGalaxy)
  gui.addColor(paramMeters, 'insideColor').onFinishChange(generateGalaxy)
  gui.addColor(paramMeters, 'ousideColor').onFinishChange(generateGalaxy)
  
  
  /**
   * Sizes
   */
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
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
    camera.position.x = 3
    camera.position.y = 3
    camera.position.z = 3
    scene.add(camera)
  
    const controls = new OrbitControls(camera, scene_ref.value)
    controls.enableDamping = true
  
    const renderer = new THREE.WebGLRenderer({
        canvas: scene_ref.value
    })
    renderer.setSize(sizes.width, sizes.height)
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  
    /**
     * Animate
     */
    const clock = new THREE.Clock()
  
    const tick = () => {
      controls.update()
      renderer.render(scene, camera)
      window.requestAnimationFrame(tick)
    }
    tick()
  })
  
  </script>
  
  <script></script>