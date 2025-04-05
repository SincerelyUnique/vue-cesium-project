<template>
  <div ref="cesiumContainer" class="cesium-viewer"></div>
</template>

<script setup>
import {onMounted, onUnmounted, ref, nextTick} from 'vue';
import {
  Viewer,
  createWorldTerrainAsync,  // 使用正确的异步地形创建方法
  ArcGisMapServerImageryProvider,
  Cartesian3,
  Math as CesiumMath,
  IonResource,
  Credit,
  Rectangle,
  Resource,
  Color,
  LabelStyle,
  VerticalOrigin,
  Cartesian2,
} from 'cesium';
import 'cesium/Build/Cesium/Widgets/widgets.css';

const cesiumContainer = ref(null);
let viewer = null;

onMounted(async () => {
  try {
    // 1. 确保DOM完全渲染
    await nextTick();

    if (!cesiumContainer.value) {
      throw new Error('Cesium container element not found');
    }

    // 2. 创建地形提供者 (异步方式)
    const terrainProvider = await createWorldTerrainAsync({
      requestVertexNormals: true,
      requestWaterMask: true
    });

    // 3. 初始化Viewer
    viewer = new Viewer(cesiumContainer.value, {
      terrain: undefined,
      imageryProvider: false, // 禁用默认影像
      // 禁用所有控件
      animation: false,  // 动画控件
      baseLayerPicker: true,  // 底图切换器
      fullscreenButton: true,  // 全屏按钮
      vrButton: false,
      geocoder: true,  // 右上角的搜索框（Geocoder）
      homeButton: true,   // 返回视角按钮
      infoBox: false,  // 信息框
      sceneModePicker: true,
      selectionIndicator: false,
      timeline: true,  // 隐藏时间线控件
      navigationHelpButton: true,  //帮助按钮
    });

    // 4. 直接设置地形提供者（不再需要readyEvent监听）
    viewer.scene.terrainProvider = terrainProvider;

    // 5. 添加影像图层
    let arcgisProvider;
    try {
      arcgisProvider = await ArcGisMapServerImageryProvider.fromUrl(
          'https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer',
          {
            enablePickFeatures: false,
            credit: new Credit('ArcGIS World Imagery'),
            maximumLevel: 19,
            rectangle: Rectangle.fromDegrees(-180, -90, 180, 90)
          }
      );

      // 确保资源存在
      if (!arcgisProvider.resource) {
        arcgisProvider.resource = new Resource({
          url: 'https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer'
        });
      }
    } catch (error) {
      console.error('Failed to create ArcGIS provider:', error);
      // 回退到 Ion 影像
      arcgisProvider = await IonResource.fromAssetId(3812);
    }
    viewer.imageryLayers.addImageryProvider(arcgisProvider);

    // 5. 设置初始视角
    viewer.camera.flyTo({
      destination: Cartesian3.fromDegrees(117.2272, 31.8206, 1000000),
      orientation: {
        heading: CesiumMath.toRadians(0),
        pitch: CesiumMath.toRadians(-45),
        roll: 0.0
      }
    });

    const cities = [
      { name: '北京市', lon: 116.4, lat: 39.9 },
      { name: '上海市', lon: 121.47, lat: 31.23 },
      { name: '合肥市', lon: 117.2272, lat: 31.8206 }
    ];
    cities.forEach(city => {
      viewer.entities.add({
        name: city.name,
        position: Cartesian3.fromDegrees(city.lon, city.lat),
        label: {
          text: city.name,
          font: '14pt sans-serif',
          fillColor: Color.WHITE,
          outlineColor: Color.BLACK,
          outlineWidth: 2,
          style: LabelStyle.FILL_AND_OUTLINE,
          verticalOrigin: VerticalOrigin.BOTTOM,
          pixelOffset: new Cartesian2(0, -20) // 调整标签位置
        }
      });
    });

  } catch (error) {
    console.error('Cesium initialization error:', error);
  }
});

onUnmounted(() => {
  if (viewer && !viewer.isDestroyed()) {
    viewer.destroy();
  }
});
</script>

<style scoped>
.cesium-viewer {
  width: 100%;
  height: calc(100vh - 75px);
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>