# FristGame 项目结构速览

**Godot 4.2+** · **`snake_case`** · 主场景 **`res://bootstrap/main.tscn`** · 图标 **`res://icon.svg`**

| 区域 | 路径 |
|------|------|
| 启动入口 | `bootstrap/` |
| 媒体与美术归口 | `assets/`（材质统一 **`assets/materials/`**；无 `assets/3d/materials/`） |
| 表与清单源 | `data/` |
| Godot 序列化资源 | `resources/`（配置实例在 **`resources/configs/`**） |
| 游戏场景 | **`scenes/`** 子域：`characters`、`enemies`、`items`、`levels`、`ui`、`vfx` |
| 游戏业务脚本 | **`scripts/game/`** 子域：**与 `scenes/` 同名同级**，一一镜像 |
| 全局单例与管理器脚本 | **`scripts/autoload/`**（多文件、单职责） |
| 业务枚举与常量 | **`scripts/common/enums/`**、**`constants/`** |
| 模式与基础设施 | **`scripts/core/architecture/`** |
| 通用 / 抽象基类 | **`scripts/core/global_base/`**（仅基类；无枚举、常量、业务逻辑） |
| 原生类型扩展 | **`scripts/utilities/extensions/`** |
| 纯函数工具 | **`scripts/utilities/helpers/`**（数学、字符串、随机等） |
| 分包 / 热更 / 远程缓存代码 | **`scripts/resource_loader/`** |
| 本地化资源 | `localization/locales/` |
| 测试 | `tests/unit/`、`tests/integration/` |

**镜像速查**：`scenes/<域>/` ↔ `scripts/game/<域>/`（`<域>` 为上表所列子目录名）。

完整树与约束 → [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
