<template>
  <div ref="cesiumContainer" class="cesium-viewer"></div>
</template>

<script setup>
import {onMounted, onUnmounted, ref, nextTick} from 'vue';
import {
  Viewer,
  createWorldTerrainAsync,  // 使用正确的异步地形创建方法
  ArcGisMapServerImageryProvider,
  WebMapTileServiceImageryProvider,
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
  WebMercatorTilingScheme,
  Ion,
} from 'cesium';
import 'cesium/Build/Cesium/Widgets/widgets.css';

const cesiumContainer = ref(null);
let viewer = null;

onMounted(async () => {
  try {
    const tdtKey = ""
    const cesiumAsset = ""
    let subdomains=['0','1','2','3','4','5','6','7'];
    Ion.defaultAccessToken = cesiumAsset;

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
      animation:false,       //动画
      homeButton:true,       //home键
      geocoder:true,         //地址编码
      baseLayerPicker:true, //图层选择控件
      timeline:false,        //时间轴
      fullscreenButton:true, //全屏显示
      infoBox:true,         //点击要素之后浮窗
      sceneModePicker:true,  //投影方式  三维/二维
      navigationInstructionsInitiallyVisible:false, //导航指令
      navigationHelpButton:false,     //帮助信息
      selectionIndicator:false, // 选择
      imageryProvider: new WebMapTileServiceImageryProvider({
        //影像底图
        url: "http://t{s}.tianditu.com/img_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=img&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default&format=tiles&tk=" + tdtKey,
        subdomains: subdomains,
        layer: "tdtImgLayer",
        style: "default",
        format: "image/jpeg",
        tileMatrixSetID: "GoogleMapsCompatible",//使用谷歌的瓦片切片方式
        show: true
      })
    });

    // 4. 直接设置地形提供者（不再需要readyEvent监听）
    viewer.scene.terrainProvider = terrainProvider;

    // 5. 添加影像图层
    //viewer._cesiumWidget._creditContainer.style.display = "none";  // 隐藏cesium ion
    viewer.imageryLayers.addImageryProvider(new WebMapTileServiceImageryProvider({
      //影像注记
      url: "http://t{s}.tianditu.com/cia_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=cia&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default.jpg&tk=" + tdtKey,
      subdomains: subdomains,
      layer: "tdtCiaLayer",
      style: "default",
      format: "image/jpeg",
      tileMatrixSetID: "GoogleMapsCompatible",
      show: true
    }));
    //viewer.imageryLayers.addImageryProvider(arcgisProvider);

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