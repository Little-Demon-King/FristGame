# FristGame 完整架构说明

本文档描述 **Godot 4.2+** 工程目录、模块职责、依赖约定与扩展方式；与仓库根目录 **`PROJECT_STRUCTURE.md`**（速览）配套使用，目录变更时请同步更新两处。

---

## 1. 设计目标

| 目标 | 落地方式 |
|------|------------|
| 低耦合、高内聚 | 跨模块依赖 `contracts` 与事件契约；协调用 `mediators` / `event_bus`，避免场景脚本互相硬引用 |
| 数据与表现分离 | 数值与表：`data/`、`resources/configs/`；表现：`scenes/`；行为：`scripts/game/` + `core` |
| 适配 Godot 常规结构 | 主场景在 **`bootstrap/`**；Autoload 在 **`scripts/autoload/`** 顶层；不做 DDD 四层目录 |
| 兼顾 C# | 命名空间与 `scripts/` 目录层级一致；`presentation_patterns/mvvm` 服务 UI 绑定习惯 |
| 可长期演进 | 配置与存档带版本；分包与热更清单独立；调试与日志统一入口 |

---

## 2. 路径歧义说明（必读）

| 路径 | 性质 | 用途 |
|------|------|------|
| **`res://resources/`** | Godot 约定资源根 | `.tres` / `.res` 等序列化资源；子目录 **`configs/`** 存配置型 Resource 与表解析结果 |
| **`res://scripts/resources/`** | 逻辑代码目录 | 分包加载、热更新管线、远程缓存策略等与 **引擎资源根** 无关，禁止与前者混称 |

---

## 3. 标准目录树（含用途）

```
FristGame/
├── bootstrap/                                   # 唯一启动入口：主场景；可选挂载全局调试、DI 组合根
├── addons/                                      # 引擎插件目录（固定名）
│   └── plugin_modules/                          # 按契约加载的业务/功能插件包宿主（非引擎本体）
├── assets/                                      # 美术与媒体统一归口；.import 由引擎维护
│   ├── 2d/
│   │   ├── textures/                            # 通用贴图、精灵表、UI 图素
│   │   ├── sprites/                             # 按对象拆分的精灵源
│   │   ├── tilesets/                            # TileMap 地砖源图
│   │   └── vfx/                                 # 粒子与全屏特效用图
│   ├── 3d/
│   │   ├── meshes/                              # 模型源（如 .glb）
│   │   ├── textures/                            # 3D 贴图
│   │   └── materials/                           # 3D 材质资源
│   ├── audio/
│   │   ├── sfx/                                 # 短音效
│   │   └── music/                               # BGM、章节音乐
│   ├── fonts/                                   # 字体
│   ├── animations/                              # AnimationLibrary 等共享动画资源
│   ├── materials/                               # CanvasItemMaterial / ShaderMaterial 等
│   └── prefabs/                                 # 可复用场景片段（与 scenes 内实例配合）
├── data/
│   ├── config_tables/                           # 配置表源文件（csv/json 等）
│   └── pack_manifests/                          # 分包与热更清单（包 id、依赖、哈希、URL 模板）
├── localization/
│   └── locales/                                 # 翻译资源（.po、.csv 等）
├── resources/                                   # Godot 序列化资源根（引擎约定目录名）
│   └── configs/                                 # Resource 配置、表解析后的 .tres/.res
├── scenes/                                      # 游戏内场景（不含 bootstrap）
│   ├── levels/                                  # 关卡 / 地图
│   ├── characters/                              # 玩家、NPC 根场景
│   ├── enemies/                                 # 敌人根场景
│   ├── items/                                   # 拾取物、机关、交互物
│   ├── ui/                                      # 菜单、HUD、弹窗
│   ├── vfx/                                     # 纯表现场景
│   └── presentation/                            # 大型 UI/表现组合壳（可选）
├── scripts/
│   ├── autoload/                                # Autoload 脚本：全局单例，薄层装配与转发
│   ├── core/
│   │   ├── state_machine/                       # 状态机：状态基类、转换、共享上下文
│   │   ├── components/                        # 组件化：组件接口、组合根、生命周期适配
│   │   ├── event_bus/                         # 事件总线：订阅、通道、调试钩子
│   │   ├── object_pool/                       # 对象池：策略、节点池、池化对象契约
│   │   ├── dependency_injection/              # DI：容器、注册表、作用域（与 bootstrap/autoload 协作）
│   │   ├── contracts/                         # 接口与抽象：跨模块唯一稳定依赖面
│   │   ├── commands/                          # 命令模式：命令对象、撤销/重做、批处理
│   │   ├── mediators/                         # 中介者：跨系统编排，减少 UI↔玩法 直连
│   │   ├── enums_and_constants/               # 枚举、常量、分组常量，禁止魔法数散落
│   │   └── modular_plugins/                   # 插件模块契约：发现、加载、启停生命周期
│   ├── managers/                                # 全局管理器拆分：音频、输入、场景流、时间缩放等
│   ├── configuration/
│   │   ├── runtime/                           # 运行时配置模型、合并结果、只读快照
│   │   └── hot_reload/                        # 表变更监听、校验失败回滚、通知订阅方
│   ├── persistence/
│   │   ├── saves/                             # 存档序列化、槽位、元数据
│   │   └── migrations/                        # 存档版本迁移，保证旧档可读
│   ├── localization/                          # 本地化服务：键规约、回退语言、与 locales 绑定
│   ├── debug/                                   # 全局日志（级别/分类/输出器）与未捕获异常处理
│   ├── resources/                               # 分包、热更、远程缓存（代码，非根 resources/）
│   │   ├── packs/                             # 分包加载、依赖解析，消费 pack_manifests
│   │   └── remote_cache/                      # 下载、校验、user:// 缓存路径与清理策略
│   ├── presentation_patterns/
│   │   ├── mvc/                               # MVC 基类与约定
│   │   └── mvvm/                              # ViewModel、绑定适配（C# 友好）
│   ├── utilities/
│   │   └── extensions/                        # 数学、路径、集合、Node 安全扩展等工具
│   └── game/
│       ├── characters/                        # 与 scenes/characters 对应的业务脚本
│       ├── enemies/                           # 与 scenes/enemies 对应
│       ├── items/                             # 与 scenes/items 对应
│       ├── levels/                            # 与 scenes/levels 对应
│       └── ui/                                # 与 scenes/ui 对应
├── tests/
│   ├── unit/                                  # 纯逻辑、core、无头依赖测试
│   └── integration/                           # 场景级、事件总线、存档与 IO 集成测试
├── docs/
│   └── ARCHITECTURE.md                        # 本文件
├── project.godot
├── icon.svg
└── PROJECT_STRUCTURE.md                       # 结构速览与文档入口
```

**场景与脚本对应**：`scenes/<域>/` 与 `scripts/game/<域>/` 成对扩展，避免业务脚本散落在 `scripts/` 根下。

---

## 4. 启动与全局生命周期

1. **`project.godot`**：`application/run/main_scene` → **`res://bootstrap/main.tscn`**。  
2. **`bootstrap/`**：负责最早期的组合（可选：注册 DI、挂接 `debug` 里日志/异常处理器、拉取首包配置）。  
3. **`scripts/autoload/`**：Project Settings → Autoload 指向的脚本；**仅做**解析、注册、生命周期转发，**不写**大块业务逻辑。  
4. **`scripts/managers/`**：可具体实现的单职责全局服务（音频、输入等），由 Autoload 或 DI 持有接口实现。

---

## 5. 数据流（配置表驱动 + 热重载）

```
data/config_tables/（源表）
        ↓ 加载 / 校验
scripts/configuration/runtime/（运行时只读模型）
        ↓ 可选序列化快照
resources/configs/（.tres 等 Resource 实例）
        ↓ 供读
scripts/game/* 与 scenes/*（表现与玩法，只读配置接口）
```

- **`scripts/configuration/hot_reload/`**：监听 `data/config_tables/` 变更 → 重新校验 → 更新 `runtime` 快照 → 通过事件或接口通知订阅方；失败时回滚并保持上次一致快照。  
- **禁止**：在 UI 或角色脚本内直接解析 csv 路径字符串；统一走 configuration 管线。

---

## 6. 持久化与版本

- **`scripts/persistence/saves/`**：写入/读取用户数据；头信息中带 **格式版本号**。  
- **`scripts/persistence/migrations/`**：按版本链式迁移；新增字段用默认值补全，**禁止**静默改变旧字段语义。  
- 与配置表版本独立：存档版本只描述存档 blob，不替代 `data/config_tables` 的表版本字段。

---

## 7. 分包与热更新

- **`data/pack_manifests/`**：描述包列表、依赖、校验信息、远程基地址模板。  
- **`scripts/resources/packs/`**：根据清单加载 PCK/补丁、解析依赖顺序、与 Godot 加载 API 对接。  
- **`scripts/resources/remote_cache/`**：下载、哈希校验、`user://` 下缓存目录约定与清理；不提交用户缓存到版本库。

---

## 8. 本地化

- **`localization/locales/`**：各语言翻译文件。  
- **`scripts/localization/`**：`TranslationServer` 封装、键命名规范、缺键回退、占位符替换；**不**在控件脚本内硬编码长句。

---

## 9. 调试（日志 + 异常）

- **`scripts/debug/`**：分级日志、分类（渠道）、输出目标（文件/控制台/远程）；以及全局未捕获异常回调、致命错误提示与上报钩子。  
- **注册点**：在 **`bootstrap`** 最早阶段或 **Autoload** 初始化中完成，保证后续系统可安全打日志。

---

## 10. 架构能力映射表

### 10.1 第一档（刚需 8 项）

| 能力 | 目录 |
|------|------|
| 状态机 | `scripts/core/state_machine/` |
| 组件化 | `scripts/core/components/` |
| 事件总线 | `scripts/core/event_bus/` |
| 数据与表现分离 | `data/`、`resources/configs/`、`scenes/`、`scripts/game/` |
| 全局管理器拆分 | `scripts/managers/` + `scripts/autoload/`（装配） |
| 配置表驱动 | `data/config_tables/` + `scripts/configuration/runtime/` + `resources/configs/` |
| 枚举与常量中心化 | `scripts/core/enums_and_constants/` |
| 适度分层 | `core`（横切能力）+ `game`（业务）+ `bootstrap`（入口），无四层 DDD 目录 |

### 10.2 第二档（进阶 10 项）

| 能力 | 目录 |
|------|------|
| 接口契约 | `scripts/core/contracts/` |
| 对象池 | `scripts/core/object_pool/` |
| 依赖注入 | `scripts/core/dependency_injection/` + `bootstrap/` |
| 模块化 / 插件化 | `scripts/core/modular_plugins/` + `addons/plugin_modules/` |
| 单例职责拆分 | `scripts/managers/`（一文件一职责） |
| MVC / MVVM | `scripts/presentation_patterns/mvc`、`mvvm` + `scenes/ui/` |
| 命令模式 | `scripts/core/commands/` |
| 中介者模式 | `scripts/core/mediators/` |
| 通用扩展与工具 | `scripts/utilities/extensions/` |
| 存档版本兼容 | `scripts/persistence/migrations/` + `saves/` |

### 10.3 额外高级（4 项）

| 能力 | 目录 |
|------|------|
| 资源分包 + 热更新 | `data/pack_manifests/` + `scripts/resources/packs/` + `remote_cache/` |
| 多语言 | `localization/locales/` + `scripts/localization/` |
| 全局日志 + 全局异常 | `scripts/debug/` |
| 配置表热重载 | `scripts/configuration/hot_reload/` |

---

## 11. 依赖与工程约束

1. **`scripts/autoload/`**：禁止堆积业务；复杂逻辑下放 `game/` 或 `managers/`。  
2. **跨模块引用**：优先 `contracts` 与事件载荷类型；避免 `game` 子域之间循环引用具体类。  
3. **`core`**：不依赖具体关卡或 UI 场景路径；可由 `bootstrap` 或 `game` 反向使用 `core`。  
4. **配置与存档**：变更需兼容策略（表版本字段 + 解析分支；存档 `migrations`）。  
5. **C#**：程序集名与 `project.godot` 中 `dotnet/project/assembly_name` 一致；公共类型优先放在 `contracts` 与 `enums_and_constants`。

---

## 12. 测试

| 目录 | 用途 |
|------|------|
| `tests/unit/` | 纯函数、状态机、配置解析、迁移逻辑单测 |
| `tests/integration/` | 需场景树或 Autoload 的集成用例、事件总线联调 |

测试工程（GUT、gdUnit4 等）的插件仍放在 **`addons/`**，与本表目录不冲突。

---

## 13. 文档维护

新增或废弃目录、调整 Autoload 与主场景路径时，必须更新：

- **`docs/ARCHITECTURE.md`**（本文件）  
- **`PROJECT_STRUCTURE.md`**（速览表）

---

*文档版本与仓库目录结构一致；若编辑器缓存中仍出现已废弃路径（如 `scenes/bootstrap`、`resources/definitions`），以本文件 §3 为准并清理残留。*
