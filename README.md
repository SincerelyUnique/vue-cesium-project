# vue-cesium-project

> learn vue-cesium, project init from https://github.com/element-plus/element-plus-vite-starter

## dependency

- node 22.14.0

## install cesium
npm install cesium @cesium/engine @cesium/widgets vite-plugin-cesium -D

## vite.config.js
```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import cesium from 'vite-plugin-cesium'

export default defineConfig({
  plugins: [vue(), cesium()]
})
```

## reference

https://github.com/element-plus/element-plus-vite-starter
https://github.com/CesiumChina/cesium-map/tree/master
https://console.tianditu.gov.cn/api/person 天地图key
https://ion.cesium.com/tokens?page=1 cesium default token
