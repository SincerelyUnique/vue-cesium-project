# vue-cesium-project

> learn vue-cesium

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


