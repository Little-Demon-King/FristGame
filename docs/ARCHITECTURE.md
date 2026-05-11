# FristGame 完整架构说明

本文档描述 **Godot 4.2+** 工程目录、模块职责、依赖约定与扩展方式；与仓库根目录 **`PROJECT_STRUCTURE.md`**（速览）配套使用，目录变更时请同步更新两处。

---

## 1. 设计目标

| 目标 | 落地方式 |
|------|------------|
| 低耦合、高内聚 | 跨模块依赖 `architecture/contracts` 与事件契约；协调用 `mediators` / `event_bus` |
| 数据与表现分离 | 数值与表：`data/`、`resources/configs/`；表现：`scenes/`；行为：`scripts/game/` + `core` |
| 架构与业务解耦 | 模式与基础设施在 **`scripts/core/`**；**全局业务枚举与常量**在 **`scripts/common/`** |
| 适配 Godot 常规结构 | 主场景 **`bootstrap/`**；Autoload **`scripts/autoload/`**（含原「全局管理器」职责的拆分单例脚本） |
| 兼顾 C# | 命名空间与 `scripts/` 目录层级一致；`presentation_patterns/mvvm` 服务 UI 绑定习惯 |
| 可长期演进 | 配置与存档带版本；分包与热更清单独立；调试统一入口 |

---

## 2. 路径说明（必读）

| 路径 | 性质 | 用途 |
|------|------|------|
| **`res://resources/`** | Godot 约定资源根 | `.tres` / `.res`；子目录 **`configs/`** 存配置型 Resource |
| **`res://scripts/resource_loader/`** | 逻辑代码 | 分包、热更、远程缓存加载管线（**不与**根目录 `resources/` 同名，避免混淆） |
| **`res://assets/materials/`** | 美术资源 | **2D / 3D 共用**材质与着色资源归口（不再使用 `assets/3d/materials/`） |

---

## 3. 标准目录树（含用途）

```
FristGame/
├── bootstrap/                                   # 唯一启动入口：主场景；全局装配
├── addons/
│   └── plugin_modules/                          # 契约化功能插件宿主
├── assets/                                      # 美术与媒体统一归口
│   ├── 2d/
│   │   ├── textures/                            # 通用贴图、精灵表、UI 图素
│   │   ├── sprites/                             # 按对象拆分的精灵源
│   │   ├── tilesets/                            # TileMap 地砖源图
│   │   └── vfx/                                 # 粒子与全屏特效用图
│   ├── 3d/
│   │   ├── meshes/                              # 模型源（如 .glb）
│   │   └── textures/                            # 3D 贴图（材质统一见 assets/materials/）
│   ├── audio/
│   │   ├── sfx/                                 # 短音效
│   │   └── music/                               # BGM
│   ├── fonts/                                   # 字体
│   ├── animations/                              # AnimationLibrary 等共享动画资源
│   ├── materials/                               # 2D/3D 材质与 ShaderMaterial（全项目唯一材质归口）
│   └── prefabs/                                 # 可复用场景片段资源
├── data/
│   ├── config_tables/                           # 配置表源
│   └── pack_manifests/                          # 分包与热更清单
├── localization/
│   └── locales/                                 # 翻译文件
├── resources/
│   └── configs/                                 # Resource 配置与表解析结果
├── scenes/                                      # 游戏内场景（与 scripts/game 子域一一镜像）
│   ├── characters/                              # 玩家、NPC 根场景
│   ├── enemies/                                 # 敌人根场景
│   ├── items/                                   # 拾取物、机关、交互物
│   ├── levels/                                  # 关卡 / 地图场景
│   ├── ui/                                      # 菜单、HUD、弹窗、大型 UI 壳（子目录或独立 .tscn）
│   └── vfx/                                     # 纯表现场景
├── scripts/
│   ├── autoload/                                # Autoload：全局单例；音频 / 场景流 / 输入等一脚本一职责
│   ├── common/
│   │   ├── enums/                               # 全局业务枚举
│   │   └── constants/                           # 全局业务常量
│   ├── core/
│   │   ├── architecture/                        # 可复用模式与横切基础设施
│   │   │   ├── state_machine/
│   │   │   ├── components/
│   │   │   ├── event_bus/
│   │   │   ├── object_pool/
│   │   │   ├── dependency_injection/
│   │   │   ├── contracts/
│   │   │   ├── commands/
│   │   │   ├── mediators/
│   │   │   └── modular_plugins/
│   │   └── global_base/                         # 仅基类：见 §10，禁止枚举 / 常量 / 业务逻辑
│   ├── configuration/
│   │   ├── runtime/                             # 运行时配置模型与快照
│   │   └── hot_reload/                          # 配置表热重载
│   ├── persistence/
│   │   ├── saves/                               # 存档读写
│   │   └── migrations/                          # 存档版本迁移
│   ├── localization/                            # 本地化服务代码
│   ├── debug/                                   # 全局日志与异常
│   ├── resource_loader/
│   │   ├── packs/                               # 分包加载
│   │   └── remote_cache/                        # 远程补丁与缓存策略
│   ├── presentation_patterns/
│   │   ├── mvc/
│   │   └── mvvm/
│   ├── utilities/
│   │   ├── extensions/                          # 引擎 API / 原生类型扩展方法
│   │   └── helpers/                             # 数学、字符串、随机、业务无关纯函数工具
│   └── game/                                    # 业务脚本（与 scenes 子域同名同层级）
│       ├── characters/                          # 对应 scenes/characters/
│       ├── enemies/                             # 对应 scenes/enemies/
│       ├── items/                               # 对应 scenes/items/
│       ├── levels/                              # 对应 scenes/levels/
│       ├── ui/                                  # 对应 scenes/ui/
│       └── vfx/                                 # 对应 scenes/vfx/
├── tests/
│   ├── unit/
│   └── integration/
├── docs/
│   └── ARCHITECTURE.md
├── project.godot
├── icon.svg
└── PROJECT_STRUCTURE.md
```

### 3.1 场景与脚本镜像（对齐维护）

| `scenes/` | `scripts/game/` |
|-----------|-----------------|
| `scenes/characters/` | `scripts/game/characters/` |
| `scenes/enemies/` | `scripts/game/enemies/` |
| `scenes/items/` | `scripts/game/items/` |
| `scenes/levels/` | `scripts/game/levels/` |
| `scenes/ui/` | `scripts/game/ui/` |
| `scenes/vfx/` | `scripts/game/vfx/` |

新增域时两边同步加同名子目录；查找、维护、复用路径一致。

---

## 4. 启动与全局生命周期

1. **`project.godot`**：`run/main_scene` → **`res://bootstrap/main.tscn`**。  
2. **`bootstrap/`**：最早组合（DI、调试钩子、首包配置等）。  
3. **`scripts/autoload/`**：每个 Autoload 脚本 **单一职责**；原独立 `managers/` 中的能力（音频、输入、场景流等）以 **多个 Autoload 单例脚本** 或 **同一装配脚本内组合多个小管理器类型** 形式存在于此目录，**禁止**单文件上帝类。  
4. **`scripts/core/architecture/`**：可被 Autoload 与 `game` 引用的模式实现；**不**写具体关卡剧情数值。

---

## 5. 数据流（配置表驱动 + 热重载）

```
data/config_tables/
        ↓
scripts/configuration/runtime/
        ↓
resources/configs/
        ↓
scripts/game/* 与 scenes/*
```

热重载、禁止直读表路径等约定不变，见 `scripts/configuration/hot_reload/`。

---

## 6. 持久化与版本

- **`scripts/persistence/saves/`**、**`migrations/`**：版本头与链式迁移；规则同前。

---

## 7. 分包与热更新

- **`data/pack_manifests/`**  
- **`scripts/resource_loader/packs/`**、**`remote_cache/`**：加载、依赖、下载与 `user://` 缓存策略。

---

## 8. 本地化

- **`localization/locales/`** + **`scripts/localization/`**

---

## 9. 调试（日志 + 异常）

- **`scripts/debug/`**；在 `bootstrap` / Autoload 早期注册。

---

## 10. `core` 子目录职责

| 目录 | 职责 |
|------|------|
| **`core/architecture/`** | 状态机、组件、事件总线、对象池、DI、契约、命令、中介者、模块化插件等 **可复用模式与基础设施** |
| **`core/global_base/`** | **仅**放各类 **通用基类** 与 **抽象基类**：如状态基类、实体基类、UI 基类、自定义 `Node`/`Node2D`/`Control` 基类等。**禁止**放入：业务枚举、业务常量、具体业务逻辑、关卡/数值脚本；枚举与常量归 **`scripts/common/`**，玩法逻辑归 **`scripts/game/`**。 |

### 10.1 `utilities`：`extensions` 与 `helpers` 分离

| 目录 | 职责 |
|------|------|
| **`utilities/extensions/`** | 对 **Godot / C# 引擎原生类型** 的扩展方法或薄封装（如 `Vector2`、`Node`、`string` 的链式扩展）。 |
| **`utilities/helpers/`** | **与引擎类型无绑定**的纯工具：数学、字符串、随机、格式化、集合算法等；可含项目内通用纯函数，**不**塞业务状态。 |

---

## 11. `common` 与架构边界

- **`scripts/common/enums/`**、**`constants/`**：仅放 **玩法、关卡、UI 业务域** 的枚举与常量，**不**依赖 `core/architecture` 内具体类（可依赖 Godot 内置类型）。  
- **`core/architecture/contracts/`**：跨模块 **接口与抽象类型**；业务枚举如需出现在接口签名中，类型定义仍建议在 **`common`**，由 `contracts` 引用。  
- **禁止**在 `core/architecture` 内堆积关卡名、波次 ID 等业务枚举（迁至 **`common`**）。

---

## 12. 架构能力映射表

### 12.1 第一档（刚需 8 项）

| 能力 | 目录 |
|------|------|
| 状态机 | `scripts/core/architecture/state_machine/` |
| 组件化 | `scripts/core/architecture/components/` |
| 事件总线 | `scripts/core/architecture/event_bus/` |
| 数据与表现分离 | `data/`、`resources/configs/`、`scenes/`、`scripts/game/` |
| 全局管理器（拆分） | **`scripts/autoload/`**（多单例脚本，一职责一条目） |
| 配置表驱动 | `data/config_tables/` + `scripts/configuration/runtime/` + `resources/configs/` |
| 枚举与常量（业务） | **`scripts/common/enums/`**、**`constants/`** |
| 适度分层 | `core/architecture` + `core/global_base` + `common` + `game` + `bootstrap` |

### 12.2 第二档（进阶 10 项）

| 能力 | 目录 |
|------|------|
| 接口契约 | `scripts/core/architecture/contracts/` |
| 对象池 | `scripts/core/architecture/object_pool/` |
| 依赖注入 | `scripts/core/architecture/dependency_injection/` + `bootstrap/` |
| 模块化 / 插件化 | `scripts/core/architecture/modular_plugins/` + `addons/plugin_modules/` |
| 单例职责拆分 | **`scripts/autoload/`**（禁止单文件承担全部全局职责） |
| MVC / MVVM | `scripts/presentation_patterns/` + **`scenes/ui/`**（含大型 UI 壳场景） |
| 命令模式 | `scripts/core/architecture/commands/` |
| 中介者模式 | `scripts/core/architecture/mediators/` |
| 扩展与工具 | `scripts/utilities/extensions/`（原生类型扩展）、`scripts/utilities/helpers/`（数学/字符串/随机等纯函数） |
| 存档版本兼容 | `scripts/persistence/migrations/` + `saves/` |

### 12.3 额外高级（4 项）

| 能力 | 目录 |
|------|------|
| 资源分包 + 热更新 | `data/pack_manifests/` + **`scripts/resource_loader/packs/`** + **`remote_cache/`** |
| 多语言 | `localization/locales/` + `scripts/localization/` |
| 全局日志 + 全局异常 | `scripts/debug/` |
| 配置表热重载 | `scripts/configuration/hot_reload/` |

---

## 13. 依赖与工程约束

1. **`scripts/autoload/`**：可含多个轻量全局服务脚本；复杂领域逻辑在 **`scripts/game/`**。  
2. **跨模块引用**：优先 **`architecture/contracts`** 与事件载荷；业务字面量来自 **`common`**。  
3. **`core/architecture`**：不依赖具体 `.tscn` 路径；**`core`** 整体不引用 **`game`**。  
4. **配置与存档**：表版本与存档 `migrations` 策略不变。  
5. **C#**：程序集名与 `project.godot` 中 `assembly_name` 一致；对外稳定类型在 **`contracts`**，业务枚举在 **`common`**。

---

## 14. 测试

| 目录 | 用途 |
|------|------|
| `tests/unit/` | 纯逻辑、architecture、configuration、migrations |
| `tests/integration/` | Autoload、事件总线、场景级联调 |

---

## 15. 文档维护

更新目录或 Autoload 策略时，同步 **`docs/ARCHITECTURE.md`** 与 **`PROJECT_STRUCTURE.md`**。

---

*已废弃路径（勿再新增资源）：`scripts/managers/`、`scenes/presentation/`、`assets/3d/materials/`、`scripts/core/enums_and_constants/`（平铺）、`scripts/resources/`（已更名为 **`scripts/resource_loader/`**）。主场景仅使用 **`res://bootstrap/main.tscn`**。*
