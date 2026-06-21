<template>
  <div class="app-container">
    <div :class="['search-wrapper', { collapsed: !showSearch }]">
      <div v-show="showSearch" class="search-bar">
        <el-input
          v-model="searchKeyword"
          placeholder="请输入标记点"
          clearable
          style="width: 200px"
          @keyup.enter="handleSearch"
        />
        <el-button type="primary" @click="handleSearch">搜索</el-button>
        <el-button class="toggle-btn" title="隐藏搜索框" @click="showSearch = false">
          <el-icon><ArrowUp /></el-icon>
        </el-button>
      </div>
      <el-button
        v-show="!showSearch"
        class="expand-btn"
        title="显示搜索框"
        @click="showSearch = true"
      >
        <el-icon><Search /></el-icon>
      </el-button>
    </div>
    <div ref="mapContainer" class="map-container"></div>
  </div>
</template>

<script setup>
import AMapLoader from '@amap/amap-jsapi-loader'
import { ref, onMounted, onUnmounted } from 'vue'
import { ArrowUp, Search } from '@element-plus/icons-vue'

const mapContainer = ref(null)
const searchKeyword = ref('')
const showSearch = ref(false)

let map = null
let infoWindow = null
const markerList = ref([])

// 从环境变量读取高德地图配置，避免把密钥硬编码到源码中
const AMAP_KEY = import.meta.env.VITE_AMAP_KEY || ''
const AMAP_SECURITY_CODE = import.meta.env.VITE_AMAP_SECURITY_CODE || ''

// 搜索标记点并跳转
const handleSearch = () => {
  const keyword = searchKeyword.value.trim()
  if (!keyword || !map || !infoWindow) return

  const target = markerList.value.find(item => item.name.includes(keyword))
  if (!target) {
    alert('未找到该地点')
    return
  }

  map.setCenter(target.position)
  map.setZoom(15)
  infoWindow.setContent(`<div>${target.name}</div>`)
  infoWindow.open(map, target.position)
}

onMounted(() => {
  // 高德地图安全密钥，需在 .env 文件中配置
  window._AMapSecurityConfig = {
    securityJsCode: AMAP_SECURITY_CODE
  }

  // 加载高德地图 JS API
  AMapLoader.load({
    key: AMAP_KEY, // 申请好的 Web 端开发者 Key
    version: '2.0' // 指定 JS API 版本
  }).then((AMap) => {
    // 地点数据：名称 + 坐标
    const locations = [
      { name: '杭州市中心', position: [120.1551, 30.2741] },
      { name: '北京', position: [116.4074, 39.9042] }
    ]

    map = new AMap.Map(mapContainer.value, {
      zoom: 11,
      center: [120.1551, 30.2741] // 杭州
    })

    // 创建自定义信息窗体
    infoWindow = new AMap.InfoWindow({
      isCustom: true, // 使用自定义窗体
      content: '<div>HELLO,AMAP!</div>', // 信息窗体内容可以是任意 HTML 片段
      offset: new AMap.Pixel(16, -45)
    })

    // 点击 Marker 打开信息窗体
    const onMarkerClick = function (e) {
      const name = e.target.getExtData().name
      infoWindow.setContent(`<div>${name}</div>`)
      infoWindow.open(map, e.target.getPosition())
      // e.target 就是被点击的 Marker
    }

    // 创建所有标记点
    const markers = locations.map(loc => {
      const marker = new AMap.Marker({
        position: loc.position,
        extData: { name: loc.name }
      })
      marker.on('click', onMarkerClick)
      return marker
    })

    map.add(markers)
    markerList.value = locations

    // 自动调整视野，让所有标记点都显示在地图中
    map.setFitView()

    // 点击地图空白处关闭信息窗体并隐藏搜索框
    map.on('click', () => {
      infoWindow.close()
      showSearch.value = false
    })
  }).catch((e) => {
    console.error(e)
  })
})

onUnmounted(() => {
  // 销毁地图实例，释放资源
  map?.destroy()
  map = null
})
</script>

<style scoped>
.app-container {
  position: relative;
  padding: 0;
}

.search-wrapper {
  position: absolute;
  top: 12px;
  left: 12px;
  z-index: 10;
}

.search-bar {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px;
  background: rgba(255, 255, 255, 0.85);
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.search-bar :deep(.el-input__wrapper) {
  background-color: transparent;
}

.toggle-btn,
.expand-btn {
  padding: 8px 12px;
}

.expand-btn {
  background: rgba(255, 255, 255, 0.85);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.map-container {
  width: 100%;
  height: calc(100vh - 120px);
  min-height: 500px;
}
</style>
