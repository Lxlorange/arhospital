# 挂号系统-医院室内导航子系统

本项目是“医院挂号系统”小程序的一部分，主要负责**室内 3D 地图展示**与**科室路径规划**。该项目基于 Vue 2 开发，通过 Webview 嵌入到 UniApp 小程序中运行。

## 技术栈

  * **框架**: Vue 2
  * **地图引擎**: [蜂鸟云 Fengmap JavaScript SDK v3.1.10](https://www.fengmap.com/) (标准版/UMD)
  * **构建工具**: Webpack / Vue CLI
  * **通信协议**: URL Scheme (通过 URL 参数与小程序通信)

## 目录结构核心说明

```text
├── public/
│   ├── index.html          # 引入 Fengmap SDK 脚本 (Script 标签方式)
│   └── lib/                # 存放 Fengmap SDK (标准版，非 ES6)
├── src/
│   ├── components/
│   │   ├── indoorMap/
│   │   │   └── IndoorMap.vue    # 核心地图组件：负责地图初始化、搜索、画线
│   │   └── navigation/
│   │       └── NavigationBox.vue # 导航 UI 组件：起点/终点输入框
│   ├── store/              # Vuex 状态管理 (控制导航框显示等)
│   └── App.vue
└── README.md
```

## 功能特性

1.  **3D 室内地图展示**：加载医院楼层模型，支持手势缩放、旋转、楼层切换。
2.  **智能路径规划**：
      * 支持输入起点和终点名称（如“挂号大厅”、“儿科”）。
      * 自动计算最短路径并绘制路线。
      * 支持跨层导航（虽然当前业务主要在单层，但代码支持扩展）。
3.  **外部联动 (Deep Linking)**：
      * 支持通过 URL 参数直接触发导航。
      * 例如访问 `http://localhost:1234/?dest=儿科`，系统会自动弹出导航框并规划路线。
4.  **模糊搜索与自动匹配**：
      * 调用 Fengmap `FMSearchAnalyser` 接口，将科室名称自动转换为地图坐标。

## 快速开始

### 1\. 安装依赖

```bash
npm install
```

### 2\. 启动开发服务器

```bash
npm run serve
```

*默认运行在 `http://localhost:1234` (端口视配置而定)*

### 3\. 小程序集成

在 UniApp 的 `index.vue` 或 `webview.vue` 中使用：

```html
<web-view src="http://localhost:1234/?dest=儿科"></web-view>
```

## 关键开发注意事项 (踩坑指南)

如果你需要维护或修改代码，请务必阅读以下几点，这是开发过程中遇到的核心问题总结：

### 1\. SDK 版本与引入方式

  * **必须使用标准版 SDK**：本项目使用的是 **Standard (UMD/ES5)** 版本的 SDK。
  * **禁止使用 ES6 版本**：不要下载带 `(ES6)` 后缀的 SDK，也不要使用 `import { FMMap } from '...'`。
  * **引入方式**：所有 SDK 文件必须放在 `public/lib` 下，并在 `public/index.html` 中通过 `<script src="./lib/fengmap.map.min.js">` 引入。代码中直接使用全局变量 `window.fengmap`。

### 2\. 搜索接口 (v3.1.10 变更)

旧版 SDK 支持直接赋值 `request.keyword`，但 **v3.1.10** 版本必须使用 `addCondition`：

```javascript
// 正确写法
var request = new fengmap.FMSearchRequest();
request.addCondition({ keyword: '儿科' });

// 错误写法 (会导致搜索结果为空)
// request.keyword = '儿科'; 
```

### 3\. 坐标获取兼容性

不同的地图数据返回的坐标结构可能不同，为了兼容性，在 `IndoorMap.vue` 的 `searchLocation` 方法中做了如下处理：

```javascript
// 优先获取 mapCoord，其次尝试 center，最后取根节点的 x,y
const x = target.mapCoord ? target.mapCoord.x : (target.center ? target.center.x : target.x);
const y = target.mapCoord ? target.mapCoord.y : (target.center ? target.center.y : target.y);
```

### 4\. 输入框逻辑

在 `NavigationBox.vue` 中，为了防止用户修改文字后仍然使用旧的坐标缓存，添加了输入监听：

```html
<input @input="startCoord = null" ... >
```

## 待优化项

  * 考虑是否添加ar功能
  * 考虑是否添加实时定位功能