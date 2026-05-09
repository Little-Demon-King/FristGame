# FristGame 项目结构说明

引擎：**Godot 4.2+**（当前 `project.godot` 为 4.x）。主流程：**2D**（可替换为俯视角 / 射击 / 解谜 / 肉鸽，目录不必改）。

**命名**：文件夹与资源文件统一 **`snake_case`**（小写 + 下划线），与 Godot 社区惯例一致。

**入口**：`run/main_scene` → `res://scenes/bootstrap/main.tscn`；图标 → `res://icon.svg`。

---

## 目录树与用途

```
FristGame/
├── addons/                          # 插件目录（Godot 固定名）。第三方插件、编辑器工具
├── assets/                          # 媒体与字体；由编辑器生成 .import，勿手改配对文件
│   ├── 2d/                          # 2D 资源根，与 3D 导入设置隔离
│   │   ├── textures/                # 通用贴图、精灵表源图、UI 图素
│   │   ├── sprites/                 # 按角色/物体拆分的精灵相关源文件
│   │   ├── tilesets/                # TileMap 地砖、地形块源图
│   │   └── vfx/                     # 粒子贴图、屏幕特效用图
│   ├── 3d/                          # 混合项目或 3D 预览资源
│   │   ├── meshes/                  # 模型（如 .glb）
│   │   ├── textures/                # 3D 贴图
│   │   └── materials/               # 3D 侧材质资源（可选；2D 着色见根目录 materials/）
│   ├── audio/
│   │   ├── sfx/                     # 短音效
│   │   └── music/                   # BGM、章节音乐
│   └── fonts/                       # 字体（UI / HUD）
├── resources/                       # `.tres` / `.res` 数据：物品表、波次、属性模板等
│   └── definitions/                 # 自定义 Resource 的存档
├── scenes/                          # 全部 `.tscn`，按系统分域
│   ├── bootstrap/                   # 启动、加载、首屏（含 main.tscn）
│   ├── levels/                      # 关卡 / 地图场景
│   ├── characters/                  # 玩家、NPC 根场景
│   ├── enemies/                     # 敌人根场景
│   ├── items/                       # 拾取物、机关、可交互物
│   ├── ui/                          # 菜单、HUD、弹窗（Control 树）
│   └── vfx/                         # 纯表现场景（爆炸、受击反馈等）
├── scripts/                         # `.gd` / `.cs`，建议与 scenes 子域对应
│   ├── autoload/                    # 与项目设置 Autoload 单例对应的脚本（可选集中）
│   ├── core/                        # 存档、常量、事件总线等横切逻辑
│   ├── levels/                      # 关卡流、检查点、区域逻辑
│   ├── characters/                  # 移动、能力、动画状态
│   ├── enemies/                     # AI、行为
│   ├── items/                       # 拾取与效果
│   └── ui/                          # UI 逻辑
├── animations/                      # 可共享的 AnimationLibrary 等动画资源
├── materials/                       # CanvasItemMaterial / ShaderMaterial（2D 描边、后处理等）
├── prefabs/                         # 高频复用小场景（血条、掉落模板）；大体量仍归 scenes/items 等
├── tests/                           # 自动化或场景测试入口（GUT、gdUnit4 等）
├── project.godot                    # 工程配置
├── icon.svg                         # 应用图标
└── PROJECT_STRUCTURE.md             # 本说明
```

---

## 约定摘要

| 事项 | 说明 |
|------|------|
| 场景即预制体 | 可复用实体独立成 `.tscn`；小片段可放 `prefabs/` |
| 数据与美术分离 | 可调数值、表驱动内容放 `resources/`；原画音频放 `assets/` |
| 空目录 | 部分叶子目录含 `.gitkeep`，仅用于占位，可随资源增删 |

修改目录职责时，请同步更新本文件，避免协作时歧义。
