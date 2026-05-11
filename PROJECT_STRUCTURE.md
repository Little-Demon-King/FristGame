# FristGame 项目结构速览

**Godot 4.2+** · **`snake_case`** · 主场景 **`res://bootstrap/main.tscn`** · 图标 **`res://icon.svg`**

| 区域 | 路径 |
|------|------|
| 启动入口 | `bootstrap/` |
| 媒体与美术归口 | `assets/`（含 `animations/`、`materials/`、`prefabs/`） |
| 表与清单源 | `data/` |
| Godot 序列化资源 | `resources/`（配置实例在 **`resources/configs/`**） |
| 关卡与实体场景 | `scenes/`（不含 bootstrap） |
| 逻辑代码 | `scripts/`（**`autoload/`** 顶层；业务在 **`game/`**） |
| 本地化资源 | `localization/locales/` |
| 测试 | `tests/unit/`、`tests/integration/` |

完整树、模块映射、路径歧义、启动链、配置/存档/分包/本地化/调试数据流与工程约束 → [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
