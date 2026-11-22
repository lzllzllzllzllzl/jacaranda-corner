# 蓝花楹街角 · Jacaranda Corner

悉尼蓝花楹街角 **三渲二微缩场景**（Three.js / WebGL）。
奶油白三层历史建筑以圆弧转角连接两面立面，铁艺阳台、条纹雨棚、紫色木条长椅；
蓝花楹从两侧与后方围合、前低后高，花簇沿细枝分布，花瓣随风飘落；
支持 **白天 / 黄昏 / 夜晚** 切换，夜间窗户透出暖光、路灯局部照亮花簇与地面。

![模式](https://img.shields.io/badge/模式-白天·黄昏·夜晚-7b6cc0)

## 运行

**方式一（最简单）**：直接双击 `index.html`，在浏览器中打开（需联网加载 three.js CDN）。

**方式二（本地服务器）**：

```bash
cd sydney-jacaranda
python -m http.server 8000
# 打开 http://localhost:8000
```

**方式三（完全离线）**：`vendor/` 已附带 three.js r169 本地副本（`three.module.js`
与 `vendor/addons/` 下的三个官方插件）。把 `index.html` 中 importmap 的两行
CDN 地址替换为本地路径即可：

```html
<script type="importmap">
{
  "imports": {
    "three": "./vendor/three.module.js",
    "three/addons/": "./vendor/addons/"
  }
}
</script>
```

## 操作

| 操作 | 效果 |
| --- | --- |
| 左键拖拽 | 环绕旋转 |
| 滚轮 | 缩放 |
| 右键拖拽 | 平移 |
| 双击 | 复位默认机位 |
| 右下角按钮 | 白天 / 黄昏 / 夜晚切换（灯光平滑过渡） |

## 场景内容

- **方形薄底座**：双层台口 + 黄铜铭牌 "SYDNEY · JACARANDA CORNER"
- **建筑**：奶油白三层历史立面，圆弧转角（含圆窗、铜牌、吊篮），
  拱窗、店面粉窗、紫木护板、招牌 "THE JACARANDA — TEA ROOMS"，
  二层铁艺阳台（栏杆装饰环 + 紫木地板），入口条纹雨棚 + 铜壁灯，
  女儿墙后可见坡屋顶与双烟囱（陶土烟囱管）
- **街道**：转角路缘弧、斑马线、井盖、蓝紫邮筒、护柱、花箱、咖啡圆桌两椅
- **蓝花楹 ×6**：程序化生成——弯曲分叉树干（CatmullRom + Tube），
  花簇沿末梢分布、冠层留透光空隙，前低后高围合建筑
- **动态**：树梢轻摆（每棵独立相位）、150 片花瓣持续飘落、
  地面与树池积落花瓣、路灯夜景光晕与轻微闪烁

## 技术要点

- **三渲二**：全部主体使用 `MeshToonMaterial` + 4 级 `gradientMap`（RedFormat DataTexture）色阶化明暗；
  玻璃与金属件保留 `MeshStandardMaterial`（环境反射 + 金属高光），不用描边
- **时间系统**：一套目标参数表（太阳颜色/角度/强度、半球光、路灯/壁灯/窗光、花瓣色调、
  色调映射曝光、环境强度），每帧按 `1-e^(-k·dt)` 指数插值平滑过渡；页面底色由 CSS 变量同步过渡
- **程序化树**：递归分枝（4 层深度）→ TubeGeometry 合并（`mergeGeometries`）→
  花簇为带顶点色的抖动二十面体合并；树冠枢轴分层，仅冠层参与摆动
- **性能**：几何大量合并、花瓣用 `InstancedMesh`、阴影单光源 2048、
  `pixelRatio ≤ 2`、夜景仅 3 盏点光源；实测桌面端 ≈165 FPS
- **接触阴影**：实时 PCFSoft 阴影 + 径向渐变 AO 贴片双重落地感

## 文件结构

```
sydney-jacaranda/
├── index.html    ← 全部源码（HTML + CSS + JS 单文件，代码内按节注释）
├── vendor/       ← three.js r169 本地副本（离线运行用，保持 examples/jsm 目录结构）
│   ├── three.module.js
│   └── addons/
│       ├── controls/OrbitControls.js
│       ├── environments/RoomEnvironment.js
│       └── utils/BufferGeometryUtils.js
└── README.md
```

## 调参入口

打开 `index.html`，代码内均有分节注释：

- 配色 / 材质：`MAT` 对象（第 1 节）
- 三种时间的灯光参数：`MODES`（第 2 节）
- 树的位置 / 高度 / 冠幅：末尾 `makeTree` 配置数组（第 7 节）
- 花瓣数量与飘落行为：`PETAL_N`、`respawn()`（第 8 节）
- 默认机位：`HOME_P` / `HOME_T`（第 0 节）
