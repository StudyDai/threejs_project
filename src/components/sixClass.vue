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
  import real_global_01 from '/src/assets/real_global.png'
  
  const scene_ref = ref(null)
  // 设置尺寸
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }
  onMounted(() => {
    // 初始化一个gui
    const GUI = new lil.GUI()
    // 创建一个场景
    const scene = new THREE.Scene()
    // 创建一个形状
    const geometry = new THREE.BoxGeometry(1,1,1)
    // 创建一个通道 注意这个是普通纹理加载器
    const textureLoader = new THREE.TextureLoader()
    // 创建一个图片
    const real_global_img = textureLoader.load(real_global_01)
    const screw_img = textureLoader.load(demo_img)
    // 如下是立方体纹理加载器 这个是用环境贴图的时候要用到的
    const cubeTextureLoader = new THREE.CubeTextureLoader()
    // 加载上下左右 反正总共六张图如下 有顺序的 x前面 x后面 y上面 y下面 z前面 z后面  左右 上下 前后
    // const environmentMapTexture = CubeTextureLoader.load([
    //   '01.jpg',
    //   '02.jpg',
    //   '03.jpg',
    //   '04.jpg',
    //   '05.jpg',
    //   '06.jpg',
    // ])
    // 通过设置环境map设置上去
    // material.envMap = environmentMapTexture
  
    
    // 加载出来的纹理可以直接使用滤镜 例如缩小滤镜 或者放大滤镜
    // 如果使用了nearest 记得停用mipmaping
    // real_global_img.generateMipMaps = false
    // real_global_img.minFilter = THREE.NearestFilter
    // real_global_img.magFilter = THREE.NearestFilter
    // 添加普通的线性材质
    // const material = new THREE.MeshBasicMaterial({ color: 'green', wireframe: true })
    // material.color.set('red') 等同于上面的color
    // material.color = new THREE.Color('red') 等同于上面的color
    // 设置透明度 打开允许透明度
    // material.transparent = true
    // material.opacity = 0.4
    // 应用阿尔法通道进行白显黑不显的功能,记得打开transparent
    // material.alphaMap = 图片路径
    // 设置图形的可见范围,默认是只能看到前面,不可以看到背面
    // material.side = THREE.FrontSide 只看前面
    // material.side = THREE.BackSide  只看后面
    // material.side = THREE.DoubleSide 两面都可见
    // 添加纹理
    // const material = new THREE.MeshBasicMaterial({ map: texture })
    // material.map = texture 效果如上
    // 生成材质作用的物体
    // const mesh = new THREE.Mesh(geometry, material)
    
    // 使用 网格法线材质 (这个材质主要是用来检查法线是否设置正确用的,不过他默认的颜色很好看,而且帧率稳定,GPU使用率不高)
    // 为什么需要法线 因为法线会指向光源,如果法线的方向相反,那么就是阴影部分
    // const material = new THREE.MeshNormalMaterial()
  
    // 使用 自由网格材质
    // const material = new THREE.MeshMatcapMaterial()
    //  添加材质 这个材质就是让你可以尽可能的模仿没有冠光源和阴影的球,在没有光源的情况下
    // material.matcap = real_global_img
  
    // 使用 网格深度材质 这个的效果就是他会把物体染成白色 然后你远看他会模糊而且是黑色的 越近越白 可以做雾霾的效果
    // const material = new THREE.MeshDepthMaterial()
  
    // 使用 光反应材质 网络Lambert 材质 这个就是跟光源一起用的 这个线条性能还不错,不过还有更好的
    // const material = new THREE.MeshLambertMaterial()
    
    // 使用 光放射材质2 这个跟上面的一样 只不过他会让产品身上怪异的纹理消失 而且会让光线放射
    // const material = new THREE.MeshPhongMaterial()
    // 调整光泽度 越大越光滑 或者说光更集中
    // material.shininess = 1000
    // 设置高光颜色
    // material.specular = new THREE.Color('red')
  
    // 使用 网格卡通材质
    // const material = new THREE.MeshToonMaterial()
    // material.gradientMap = '传递一个渐变的图片可以看到效果'
  
    // 使用 标准网络材质
    const material = new THREE.MeshStandardMaterial()
    // 可以设置金属度 区间是0-1
    material.metalness = 0.49
    // 设置粗糙度
    material.roughness = 0.45
    // 设置材质
    material.map = screw_img
    // 添加控制面板
    GUI.add(material, 'metalness').min(0).max(1).step(0.0001)
    GUI.add(material, 'roughness').min(0).max(1).step(0.0001)
  
    material.side = THREE.DoubleSide
    // 生一个球体
    const mesh = new THREE.Mesh(
      new THREE.SphereGeometry(0.5, 16, 16), 
      material
    )
    mesh.position.x = -1.5
    // 生成一个平面
    const plane = new THREE.Mesh(
      new THREE.PlaneGeometry(1,1),
      material
    )
    // 创建一个环体
    // new THREE.TorusGeometry(
    //   radius,    // 1. 圆环的“主半径”（从中心到圆环管中心的距离）
    //   tubeRadius,// 2. 圆环“管的半径”（圆环自身管子的粗细）
    //   radialSegments, // 3. 圆环的“径向分段数”（控制圆环的圆滑程度，数值越大越圆）
    //   tubularSegments // 4. 圆环“管的分段数”（控制管子的圆滑程度，数值越大越圆）
    // )
    const torus = new THREE.Mesh(
      new THREE.TorusGeometry(0.3, 0.2, 16, 32),
      material
    )
    torus.position.x = 1.5
    // 物体添加到场景中
    scene.add(mesh, plane, torus)
    // 这个地方先粗略的使用下灯光 三维光 第一个参数是颜色 第二个参数是强度
    const ambientLight = new THREE.AmbientLight(0xffffff, 1)
    scene.add(ambientLight)
    // 创建点光源
    const pointLight = new THREE.PointLight(0xffffff, 50.5)
    // 设置点光源的位置
    pointLight.position.x = 2
    pointLight.position.y = 3
    pointLight.position.z = 4
    scene.add(pointLight)
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
  