# 游戏工程入口

此目录用于承接真实游戏工程。当前没有 Unity 工程配置或可运行场景。

## 优先接入已有成果

如果已有 Unity 工程，先备份，再将工程源码复制到 `game/UnknownCity/`；不复制旧工程的 `.git/`，不覆盖本仓库根目录的 Git 规则。保留原始文件名与 `.meta`，不要为了匹配建议目录而批量移动现有资源。

需要保留的内容：

| 工程内路径 | 作用 |
| --- | --- |
| `Assets/` 及资源、文件夹对应的 `.meta` | 场景、脚本、模型、材质、GUID 与导入设置 |
| `Packages/manifest.json` | 直接依赖 |
| `Packages/packages-lock.json` | 已解析依赖锁定结果（由 Unity 生成） |
| `Packages/` 中实际使用的嵌入包 | 工程可恢复所需的本地包源码 |
| `ProjectSettings/` | 工程设置，含 `ProjectVersion.txt` |

另行核对 `manifest.json` 中的 `file:` 本地依赖；不能依赖只有你电脑上才有的绝对路径。必要的包移入仓库可恢复的位置，许可证不允许公开分发时登记取得步骤。

`Library/`、`Temp/`、`Obj/`、`Logs/`、`UserSettings/` 和构建输出不纳入版本管理。`.gitignore` 已覆盖嵌套工程。

1. 用 `ProjectSettings/ProjectVersion.txt` 记录的编辑器版本打开工程；先保留现有渲染管线。
2. 在 Project Settings 中确认 **Version Control → Visible Meta Files**，以及 **Editor → Asset Serialization → Force Text**；菜单位置随版本可能不同。
3. 等待导入完成，保存并关闭编辑器，然后检查资源和 `.meta` 的变更。
4. 在 Git LFS 已安装、规则已生效后暂存文件；参考 [版本管理指南](../docs/VERSION_CONTROL.md)。
5. 在 Unity Hub 中添加 `game/UnknownCity/`，验证真实入口场景。不要添加仓库根目录。

如果已有的是 Blender / Maya 模型而非 Unity 工程，先放入 `art/source/models/`，登记软件版本和外部贴图；不伪造 `ProjectVersion.txt` 或 `manifest.json`。

## 需要新建 Unity 工程时

确认实际要用的编辑器版本和渲染管线后，用 Unity Hub 在 `game/` 下创建名为 `UnknownCity` 的工程。由编辑器生成 `Assets/`、`Packages/`、`ProjectSettings/`，打开并保存一次，再提交这些真实配置。

以下仅为新资源的建议组织方式，**尚未创建**；在 Unity Project 窗口内创建，以自动生成 `.meta`：

| 建议路径（相对工程根目录） | 内容 |
| --- | --- |
| `Assets/_Project/Scenes/` | 1 号楼验证场景 |
| `Assets/_Project/Scripts/` | 移动、交互等脚本 |
| `Assets/_Project/Prefabs/` | 角色、建筑模块、交互物 |
| `Assets/_Project/Art/Models/` | 引擎使用的模型导出物 |
| `Assets/_Project/Art/Textures/` | 引擎使用的贴图 |
| `Assets/_Project/Art/Materials/` | 材质 |
| `Assets/_Project/Localization/` | 中文与未来本地化资源 |

本次初始化无需锁定引擎小版本。真正生成或迁移可运行工程前，需要从已有工程确认版本；完全没有工程时再选择版本与模板。
