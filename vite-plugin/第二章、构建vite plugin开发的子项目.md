> 从这儿开始，我们进入vite plugin开发的过程，我们将在packages目录下床架core项目里面存放我们的plugins源码，需要创建一个example项目，用来调试和展示plugin项目效果

<a name="Y03qT"></a>
### 1、安装tsup
tsup是用于对ts库的打包工具，少量配置实现基础打包，用于我们插件库开发的打包足够，项目中安装tsup之后，创建tsup.config.ts文件，添加以下配置，具体的说明可以参考tsup的官方地址[https://tsup.egoist.sh/#generate-declaration-file](https://tsup.egoist.sh/#generate-declaration-file)
```typescript
import { Options } from 'tsup'

export const tsup: Options = {
  splitting: false,
  sourcemap: false,
  format: ['cjs', 'esm'],
  dts: true,
  clean: true
}
```
<a name="x0MV7"></a>
### 2、生成vite example项目
我们可以进入到workspace对应的目录下，我这边用react为例子，可以通过`yarn create vite --template react-ts` 创建一个基础的vite + react + ts的项目， 具体课本次项目的整体模板项目<br />

<a name="R9Q9v"></a>
### 3、调整core项目的package.json进行
以下是我调整的package.json样例，关于package.json说明可以参考[知乎文章](https://zhuanlan.zhihu.com/p/148089474)，这里不做过多说明
```json
{
  "name": "vite-plugin-parse-html",
  "version": "1.0.0",
  "description": "just parse html for your inject some script or css, more then inject some data",
  "main": "dist/index.js",
  "module": "dist/index.mjs",
  "types": "dist/index.d.ts",
  "keywords": [
    "vite",
    "vite-plugin",
    "vite-plugin-parse-html"
  ],
  "files": [
    "README.md",
    "README.zh-CN.md",
    "dist",
    "CHANGELOG.md"
  ],
  "scripts": {
    "dev": "npm run build --watch",
    "build": "tsup",
    "prepublishOnly": "npm run build"
  },
  "homepage": "https://github.com/KanadeHu/vite-plugin-parse-html/tree/master",
  "bugs": {
    "url": "https://github.com/KanadeHu/vite-plugin-parse-html/issues",
    "email": "zhangywwork@163.com"
  },
  "peerDependencies": {
    "vite": ">= 2.0.0"
  },
  "author": "kanade",
  "license": "ISC",
  "dependencies": {},
  "devDependencies": {
    "@types/node": "^17.0.4",
    "typescript": "^4.5.4",
    "vite": "^2.7.6"
  }
}

```
<a name="eWl6w"></a>
### 4、插件的开发过程
关于vite插件开发相关的api文档我们可以参考 [vite插件开发文档](https://vitejs.cn/guide/api-plugin.html)，下面我就开始一个插件的开发，过程不多将，此处我们不做插件方向相关的介绍，下一章我们将简单介绍一下vite插件开发需要掌握的基本前置知识点；
