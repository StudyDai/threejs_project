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
  // 导入字体加载器可以加载多个字体
  const fontLoader = new FontLoader()
  const scene_ref = ref(null)
  // 设置尺寸
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }
  // 创建一个纹理加载器
  const textureLoader = new THREE.TextureLoader()
  const matcapLoader = textureLoader.load(real_global_01)
  const matcapLoader2 = textureLoader.load(real_global_02)
  onMounted(() => {
    // 初始化一个gui
    const GUI = new lil.GUI()
    // 创建一个场景
    const scene = new THREE.Scene()
    // 直接解析已经 import 进来的 json 数据
    const font = fontLoader.parse(fontJson)
    // 创建一个坐标进去
    const axesHelper = new THREE.AxesHelper()
    // scene.add(axesHelper)
    // 因为没办法使用url的格式,所以采用另一种格式  就是先引入再解析
    const textGeometry = new TextGeometry('VMAN', {
      font,
      size: 0.5,
      depth: 0.2,
      curveSegments: 12, // 调节这个可以让三角形不是很多 提高性能 大概6左右 但是就没那么细节
      bevelEnabled: true,
      bevelThickness: 0.03, // 这个是x和y轴的斜角度 当你在调节中心位置的时候 要减掉它
      bevelSize: 0.02, // 这个是z轴的斜角度
      bevelOffset: 0,
      bevelSegments: 5  // 这个是设置圆弧度还是啥的好像,给个4就差不多了
    })
    // 获取文字的边界框 只有调用这个 这个元素的属性 boundingBox才有值
    textGeometry.computeBoundingBox()
    // boundingBox里面包含俩属性 min和max 包含的是原点偏移量的值 其实就是可以看出一个长方体
    // 然后这个长方体的各个点的组合
    // 要让他居中就是最大值的X 往左边走
    // console.log(textGeometry.boundingBox)
    // 这个调节之后 最小x和最大x应该是一样的 因为居中了，他的一半被减掉了，那么他是不是最小x会去到旁边，最大x就是最右边
    textGeometry.translate(
      -(textGeometry.boundingBox.max.x - 0.02) / 2,
      -(textGeometry.boundingBox.max.y - 0.02) / 2,
      -(textGeometry.boundingBox.max.z - 0.03) / 2
    )
    // textGeometry.center() 等同于上面的设置
    const textMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapLoader })
    // 创建一个文本几何体
    const text = new THREE.Mesh(textGeometry, textMaterial)
    scene.add(text)
    // 创建100个甜甜圈
    // 创建几何体和材质可以只进行一次 不然很浪费性能
    const donutGeometry = new THREE.TorusGeometry(0.3, 0.2, 20, 45)
    const donutMaterial = new THREE.MeshMatcapMaterial({ matcap: matcapLoader2 })
    for (let index = 0; index < 1000; index++) {
        // 应用
        const donut = new THREE.Mesh(donutGeometry, donutMaterial)
        // 设置下
        donut.position.x = (Math.random() - 0.5) * 10
        donut.position.y = (Math.random() - 0.5) * 10
        donut.position.z = (Math.random() - 0.5) * 10
        donut.rotation.x = Math.random() * Math.PI
        donut.rotation.y = Math.random() * Math.PI
        donut.rotation.z = Math.random() * Math.PI
        const scale = Math.random()
        donut.scale.set(scale, scale, scale)
        scene.add(donut)
    }
    // 下面这种方式 等打包完成就可以用
    // fontLoader.load(
    //   './assets/fonts/helvetiker_regular.typeface.json',
    //   (font) => {
    //     // 当字体加载完成就会触发这个函数
    //     const textGeometry = new TextGeometry('VMAN', {
    //       font,
    //       size: 0.5,
    //       depth: 0.2,
    //       curveSegments: 12,
    //       bevelEnabled: true,
    //       bevelThickness: 0.03,
    //       bevelSize: 0.02,
    //       bevelOffset: 0,
    //       bevelSegments: 5
    //     })
    //     const textMaterial = new THREE.MeshBasicMaterial()
    //     // 创建一个文本几何体
    //     const text = new THREE.Mesh(textGeometry, textMaterial)
    //     scene.add(text)
    //   }
    // )
    // 创建一个形状
    const geometry = new THREE.BoxGeometry(1,1,1)
    // 创建一个通道 注意这个是普通纹理加载器
    const textureLoader = new THREE.TextureLoader()
    // 创建一个图片
    const real_global_img = textureLoader.load(real_global_01)
  
    // 使用 标准网络材质
    // const material = new THREE.MeshStandardMaterial()
  
    // material.side = THREE.DoubleSide
    // 生一个球体
    // const mesh = new THREE.Mesh(
    //   new THREE.SphereGeometry(0.5, 16, 16), 
    //   material
    // )
    // mesh.position.x = -1.5
    // 物体添加到场景中
    // scene.add(mesh)
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
  