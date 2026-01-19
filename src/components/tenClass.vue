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
  // 导入字体加载器 这个是133版本之后的改动
  import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js'
  // 彩色图
  import door_color_img from './assets/textures-ghost/door/color.jpg'
  // 透明图
  import door_alpha_img from './assets/textures-ghost/door/alpha.jpg'
  // 环境遮挡图
  import door_Ambient_img from './assets/textures-ghost/door/ambientOcclusion.jpg'
  // 高度图
  import door_height_img from './assets/textures-ghost/door/height.jpg'
  // 法线图
  import door_normal_img from './assets/textures-ghost/door/normal.jpg'
  // 金属感
  import door_metalness_img from './assets/textures-ghost/door/metalness.jpg'
  // 粗糙度
  import door_roughness_img from './assets/textures-ghost/door/roughness.jpg'
  
  // 砖颜色块
  import block_color_img from './assets/textures-ghost/bricks/color.jpg'
  // 环境遮挡
  import block_ambientOcclusion_img from './assets/textures-ghost/bricks/ambientOcclusion.jpg'
  // 法线图
  import block_normal_img from './assets/textures-ghost/bricks/normal.jpg'
  // 粗糙度
  import block_roughness_img from './assets/textures-ghost/bricks/roughness.jpg'
  
  // 草地色块
  import grass_color_img from './assets/textures-ghost/grass/color.jpg'
  // 环境遮挡
  import grass_ambientOcclusion_img from './assets/textures-ghost/grass/ambientOcclusion.jpg'
  // 法线图
  import grass_normal_img from './assets/textures-ghost/grass/normal.jpg'
  // 粗糙度
  import grass_roughness_img from './assets/textures-ghost/grass/roughness.jpg'
  
  
  
  
  
  // 试验下我们的烘焙纹理吧
  const textureloader = new THREE.TextureLoader()
  // 开始导入纹理图
  const doorColorTexture = textureloader.load(door_color_img)
  const doorAlphaTexture = textureloader.load(door_alpha_img)
  const doorAmbientTexture = textureloader.load(door_Ambient_img)
  const doorHeightTexture = textureloader.load(door_height_img)
  const doorNormalTexture = textureloader.load(door_normal_img)
  const doorMetalnessTexture = textureloader.load(door_metalness_img)
  const doorRoughnessTexture = textureloader.load(door_roughness_img)
  
  // 加载砖块
  const bricksColorTexture = textureloader.load(block_color_img)
  // bricksColorTexture.colorSpace = THREE.SRGBColorSpace
  const bricksAmbientOcclusionTexture = textureloader.load(block_ambientOcclusion_img)
  const bricksNormalTexture = textureloader.load(block_normal_img)
  const bricksRoughnessTexture = textureloader.load(block_roughness_img)
  
  // 加载草地
  const grassColorTexture = textureloader.load(grass_color_img)
  const grassAmbientOcclusionTexture = textureloader.load(grass_ambientOcclusion_img)
  const grassNormalTexture = textureloader.load(grass_normal_img)
  const grassRoughnessTexture = textureloader.load(grass_roughness_img)
  // 重复加载草地
  grassColorTexture.repeat.set(8,8)
  grassAmbientOcclusionTexture.repeat.set(8,8)
  grassNormalTexture.repeat.set(8,8)
  grassRoughnessTexture.repeat.set(8,8)
  // 默认纹理不重复,所以要开启 水平
  grassColorTexture.wrapS = THREE.RepeatWrapping
  grassAmbientOcclusionTexture.wrapS = THREE.RepeatWrapping
  grassNormalTexture.wrapS = THREE.RepeatWrapping
  grassRoughnessTexture.wrapS = THREE.RepeatWrapping
  // 默认纹理不重复,所以要开启 垂直
  grassColorTexture.wrapT = THREE.RepeatWrapping
  grassAmbientOcclusionTexture.wrapT = THREE.RepeatWrapping
  grassNormalTexture.wrapT = THREE.RepeatWrapping
  grassRoughnessTexture.wrapT = THREE.RepeatWrapping
  
  const scene_ref = ref(null)
  // 设置尺寸
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }
  // 创建一个组
  const home = new THREE.Group()
  // 创建一个纹理加载器
  onMounted(() => {
    // 初始化一个gui
    const gui = new lil.GUI()
    // 创建一个场景
    const scene = new THREE.Scene()
    // 创建一个雾 2的意思就是从距离摄像机2的距离开始, 7的含义就是去到7的时候,会直接黑点,不给你看
    // 一般就是近点设置1,然后远处设置15左右差不多,就是当你距离这个场景15远的时候,直接就黑掉了啥都看不到
    const fog = new THREE.Fog('#262837', 1, 15)
    // 要设置到场景里面去
    scene.fog = fog
    // 将组添加进去
    scene.add(home)
    // 创建一个墙
    const walls = new THREE.Mesh(
      new THREE.BoxGeometry(4,2.5,4),
      new THREE.MeshStandardMaterial({
        map: bricksColorTexture,
        aoMap: bricksAmbientOcclusionTexture,
        normalMap: bricksNormalTexture,
        roughnessMap: bricksRoughnessTexture
      })
    )
    walls.geometry.setAttribute(
      'uv2',
      new THREE.Float32BufferAttribute(walls.geometry.attributes.uv.array, 2)
    )
    walls.position.y = 2.5 / 2
    home.add(walls)
    // 创建圆锥当屋顶  1是代表圆锥高度 4应该是分成多少段 3就是3个斜面 4就是4个斜面  2.5代表的边长
    const roof = new THREE.Mesh(
      new THREE.ConeGeometry(3, 1, 4),
      new THREE.MeshStandardMaterial( { color: 'red'} )
    )
    roof.position.y = 2.5 + 0.5
    roof.rotation.y = Math.PI / 4
    home.add(roof)
  
    // 创建门
    const door = new THREE.Mesh(
      new THREE.PlaneGeometry(2,2,100,100),
      new THREE.MeshStandardMaterial({
        transparent: true,
        map: doorColorTexture,
        alphaMap: doorAlphaTexture,
        aoMap: doorAmbientTexture, // 没生效是因为咩有设置door的uv坐标,如下设置
        displacementMap: doorHeightTexture, // 让门看起来立体
        displacementScale: 0.1, // 将效果缩小10倍
        normalMap: doorNormalTexture,
        metalnessMap: doorMetalnessTexture,
        roughnessMap: doorRoughnessTexture
      })
    )
    // 属性名固定uv2, 然后值是float类型的数组 Float32BufferAttribute传递两个参数,一个是数组一个是这个数组咋看,每2个作为一组数据的话
    // 就写2, 一般uv坐标都是2个一组的
    door.geometry.setAttribute(
      'uv2',
      new THREE.Float32BufferAttribute(door.geometry.attributes.uv.array, 2)
    )
    
    door.position.z = 2 + 0.01
    door.position.y = 1
    home.add(door)
  
    // 创建灌木丛
    const bush1 = new THREE.Mesh(
      new THREE.SphereGeometry(1, 16, 16),
      new THREE.MeshStandardMaterial({ color: 'skyblue' })
    )
    bush1.scale.set(0.5,0.5,0.5)
    bush1.position.set(0.8,0.2,2.2)
    
    const bush2 = new THREE.Mesh(
      new THREE.SphereGeometry(1, 16, 16),
      new THREE.MeshStandardMaterial({ color: 'skyblue' })
    )
    bush2.scale.set(0.25,0.25,0.25)
    bush2.position.set(1.4,0.1,2.1)
  
    home.add(bush1, bush2)
  
    // 设置一个墓碑组
    const graves = new THREE.Group()
    // 创建一个坟墓
    const graveGeometry = new THREE.BoxGeometry(0.6, 0.8, 0.2)
    const graveMaterial = new THREE.MeshStandardMaterial({ color: '#b2b6b1' })
    scene.add(graves)
    // 创建一个坟墓
    // const grave = new THREE.Mesh(graveGeometry, graveMaterial)
    // 创建五十个坟墓
    for (let index = 0; index < 80; index++) {
      const grave = new THREE.Mesh(graveGeometry, graveMaterial)
      const random_angle = Math.PI * 2 * Math.random()
      const radius = Math.random() * 6 + 4
      grave.position.x = Math.cos(random_angle) * radius
      grave.position.z = Math.sin(random_angle) * radius
      grave.position.y = 0.3
      grave.rotation.y = Math.PI * Math.random()
      grave.rotation.z = (Math.random() - 0.5) * 0.2
      grave.castShadow = true
      graves.add(grave)
    }
    const AxesHelper = new THREE.AxesHelper()
    // scene.add(AxesHelper)
    // 创建一个形状
    const geometry = new THREE.SphereGeometry(0.5, 32, 32)
    // 创建一个标准网格 这个材质是需要光源的 否则是看不清晰物体的
    const material = new THREE.MeshStandardMaterial()
    // 增加粗糙度
    material.roughness = 0.4
    
    // Floor
    const floor = new THREE.Mesh(
        new THREE.PlaneGeometry(20, 20),
        new THREE.MeshStandardMaterial({
          map: grassColorTexture,
          aoMap: grassAmbientOcclusionTexture,
          normalMap: grassNormalTexture,
          roughnessMap: grassRoughnessTexture
        })
    )
    floor.geometry.setAttribute(
      'uv2',
      new THREE.Float32BufferAttribute(floor.geometry.attributes.uv.array, 2)
    )
    floor.rotation.x = - Math.PI * 0.5
    floor.position.y = 0
    scene.add(floor)
  
    // camera
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
    camera.position.x = 4
    camera.position.y = 2
    camera.position.z = 5
    scene.add(camera)
  
    // 添加一个环境光
    const ambientLight = new THREE.AmbientLight('#b9d5ff', 0.12)
    gui.add(ambientLight, 'intensity').min(0).max(1).step(0.001).name('环境光强度')
    scene.add(ambientLight)
  
    // Directional light 创建一个平行光
    const moonLight = new THREE.DirectionalLight('#b9d5ff', 0.12)
    moonLight.position.set(4, 5, - 2)
    gui.add(moonLight, 'intensity').min(0).max(1).step(0.001).name('平行光强度')
    gui.add(moonLight.position, 'x').min(- 5).max(5).step(0.001)
    gui.add(moonLight.position, 'y').min(- 5).max(5).step(0.001)
    gui.add(moonLight.position, 'z').min(- 5).max(5).step(0.001)
    scene.add(moonLight)
  
    // 添加一个门口灯  1是光照强度 7是能照射到的距离
    const door_light = new THREE.PointLight('#FF7D46',3, 6)
    door_light.position.set(0, 2.2, 2.7)
    // 创建一个观察者
    const door_light_helper = new THREE.PointLightHelper(door_light, 0.1)
    door_light_helper.visible = false
    scene.add(door_light_helper)
    home.add(door_light)
  
    // 添加鬼魂
    const ghost1 = new THREE.PointLight('#ff00ff', 6, 3)
    scene.add(ghost1)
  
    const ghost2 = new THREE.PointLight('#00ffff', 6, 3)
    scene.add(ghost2)
  
    const ghost3 = new THREE.PointLight('#ffff00', 6, 3)
    scene.add(ghost3)
  
  
  
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
    // 这个是设置整个场景的颜色,可以做到你只要离得很远 你就看不到你的房子了 跟上面的fog结合使用
    renderer.setClearColor('#262837')
  
    // 集中处理阴影
    // 打开阴影图
    renderer.shadowMap.enabled = true
    // 切换阴影贴图的算法 让阴影更加的舒服
    renderer.shadowMap.type = THREE.PCFSoftShadowMap
    moonLight.castShadow = true
    door_light.castShadow = true
    ghost1.castShadow = true
    ghost2.castShadow = true
    ghost3.castShadow = true
    // 墙是可以投射的
    walls.castShadow = true
    // 草也有
    bush1.castShadow = true
    bush2.castShadow = true
    // 地面接收阴影
    floor.receiveShadow = true
  
    // 可以继续优化的 例如灯光的远近范围以及照射的长宽,创建一个helper出来
    // 设置灯光的尺寸
    door_light.shadow.mapSize.width = 256
    // 设置灯光的尺寸
    door_light.shadow.mapSize.height = 256
    // 设置最远显示距离
    door_light.shadow.camera.far = 7
  
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    const clock = new THREE.Clock()
    function tick() {
      const elapsedTime = clock.getElapsedTime()
      controls.update()
      // 渲染器渲染场景+摄像机
      renderer.render(scene, camera)
  
      // 设置鬼魂的动画
      ghost1.position.set(Math.cos(elapsedTime * 0.5)*4, Math.abs(Math.cos(elapsedTime)), Math.sin(elapsedTime * 0.5)*4)
      ghost2.position.set(Math.cos(-elapsedTime * 0.5)*5, Math.abs(Math.cos(-elapsedTime)), Math.sin(-elapsedTime * 0.5)*5)
      ghost3.position.set(Math.cos(elapsedTime * 0.32)*3, Math.abs(Math.cos(elapsedTime)), Math.sin(elapsedTime * 0.32)*3)
  
  
  
  
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
  