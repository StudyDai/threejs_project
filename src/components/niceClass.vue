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
  import bakeShadow_img from '/src/assets/bakedShadow.jpg'
  import simpleShadow_img from '/src/assets/simpleShadow.jpg'
  // 导入字体加载器 这个是133版本之后的改动
  import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js'
  // 试验下我们的烘焙纹理吧
  const textureloader = new THREE.TextureLoader()
  // 这个用了普通的阴影效果 就是假阴影
  const bakeShadowTexture = textureloader.load(bakeShadow_img)
  // 这个是采用了白线黑不显的效果
  const simpleShadowTexture = textureloader.load(simpleShadow_img)
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
    const geometry = new THREE.SphereGeometry(0.5, 32, 32)
    // 创建一个标准网格 这个材质是需要光源的 否则是看不清晰物体的
    const material = new THREE.MeshStandardMaterial()
    // 增加粗糙度
    material.roughness = 0.4
    // 环境光 放射下
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5)
    scene.add(ambientLight)
    // 方向光(定向光) 照射出来的光都是平行的 默认这个光是从正上方打向空间中心的
    const directionLight = new THREE.DirectionalLight(0xffffff, 0.4)
    // 例如把光改到右边去,那则是从右边打到空间中心去的
    directionLight.position.set(0.8, 0.45, 0.2)
    // scene.add(directionLight)
    // 只有方向光 平行光 和点光源能开启阴影 也就是说castshadow只有这三种灯光是有效的
    directionLight.castShadow = true
    // 优化阴影 阴影贴图的大小设置 切记 要是2的幂次方 因为他在预渲染的时候会进行二分
    directionLight.shadow.mapSize.width = 1024
    directionLight.shadow.mapSize.height = 1024
    // 通过设置摄像机的近点和远点如下 注意 设置照相机要先设置再渲染助手,否则不会有变化
    directionLight.shadow.camera.near = 1
    directionLight.shadow.camera.far = 4
    // 要设置边距 不然太大了, 浪费性能 而且这样子可以让阴影边缘明显改善
    // 默认设置的远近距离以及边距让你的图在阴影贴图上面显示的特别小一个玩意 你缩小之后你的产品会变大,细节更多
    directionLight.shadow.camera.left = -2
    directionLight.shadow.camera.right = 2
    directionLight.shadow.camera.top = 2
    directionLight.shadow.camera.bottom = -2
    directionLight.shadow.radius = 10 // 模糊阴影
    // const directionalLightHelper = new THREE.DirectionalLightHelper(directionLight, 0.2)
    // 这个是阴影摄像机的辅助助手,通过控制他的边界点来优化更多的瑕疵
    const directionalLightHelper = new THREE.CameraHelper(directionLight.shadow.camera)
    directionalLightHelper.visible = false
    // scene.add(directionalLightHelper)
  
    // 聚光灯 调节摄像机和平行光不一样 聚光灯传递进去的是投射 摄像机 也就是target
    const spotLight = new THREE.SpotLight(0x78ff00, 0.4, 10, Math.PI * 0.3)
    spotLight.position.set(0,2,2)
    spotLight.castShadow = true
    spotLight.shadow.mapSize.width = 1024
    spotLight.shadow.mapSize.height = 1024
    // 调节的是角度
    spotLight.shadow.camera.fov = 30
    spotLight.shadow.camera.near = 1
    spotLight.shadow.camera.far = 6
    const spotLightHelper = new THREE.CameraHelper(spotLight.shadow.camera, 0.2)
    spotLightHelper.visible = false
    // scene.add(spotLight, spotLight.target, spotLightHelper)
  
    // 点光源 
    const pointLight = new THREE.PointLight(0xffffff, 0.5)
    pointLight.position.set(-1, 1, 0)
    pointLight.castShadow = true
    // 创建助手 其实点光源是在产品的所有面进行了透视摄像机的操作 所以这个最终的一步 不一定就是从上往下看的那一个面 可能是其他面
    pointLight.shadow.mapSize.width = 1024
    pointLight.shadow.mapSize.height = 1024
    pointLight.shadow.camera.near = 0.1
    pointLight.shadow.camera.far = 5
    const pointLightHelper = new THREE.CameraHelper(pointLight.shadow.camera, 0.2)
    pointLightHelper.visible = false
    // scene.add(pointLight, pointLightHelper)
    // 应用下
    const mesh = new THREE.Mesh(geometry, material)
    // 设置我的球可以投射阴影如下设置 默认这个选项都是false
    mesh.castShadow = true
    scene.add(mesh)
    // 创建一个平面 这个是用了预阴影的
    // const plane = new THREE.Mesh(
    //   new THREE.PlaneGeometry(5, 5),
    //   new THREE.MeshBasicMaterial({
    //     map: bakeShadowTexture
    //   })
    // )
    // 这个是默认的平面
    const plane = new THREE.Mesh(
      new THREE.PlaneGeometry(5, 5),
      material
    )
    // 创建阴影平面
    const shadowPlane = new THREE.Mesh(
      new THREE.PlaneGeometry(1.5,1.5),
      new THREE.MeshBasicMaterial({
        color: 0x000000,
        transparent: true,
        alphaMap: simpleShadowTexture
      })
    )
    shadowPlane.rotation.x = -Math.PI / 2
    shadowPlane.position.y = plane.position.y - 0.4
    scene.add(shadowPlane)
    // 设置平面可以接收阴影,因为他上面放了一个球
    plane.receiveShadow = true
    plane.rotation.x = -Math.PI * 0.5
    plane.position.y = -0.5
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
    // 设置渲染器开启阴影贴图 否则没办法创建阴影
    // renderer.shadowMap.enabled = true
    // 我们要设置预处理阴影贴图的时候 是不允许开启阴影贴图的 所以吧true注释掉就行了
    // threejs 144版本之后 还要把灯光的castshadow也设置为false 否则报错
    // 拿到当前已经渲染出来多久时间了
    const clock = new THREE.Clock()
    function tick() {
      const elapsedTime = clock.getElapsedTime()
      // 让球水平移动
      mesh.position.x = Math.cos(elapsedTime) * 1.2
      mesh.position.z = Math.sin(elapsedTime) * 1.2
      mesh.position.y = Math.abs(Math.sin(elapsedTime * 3))
      // 更新阴影
      shadowPlane.position.x = mesh.position.x
      shadowPlane.position.z = mesh.position.z
      shadowPlane.material.opacity = (1 - mesh.position.y) * 0.3
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
  