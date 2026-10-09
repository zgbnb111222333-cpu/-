# Unity、模型与大文件版本管理

本指南服务于当前小型原型。先管理可恢复的源码和资源，避免把缓存、备份和构建包塞入历史。

## 1. Unity 工程

- 提交 `Assets/`、`Packages/`、`ProjectSettings/`；保留 `ProjectVersion.txt`、`manifest.json` 与生成的 `packages-lock.json`。
- 资源和对应 `.meta`（包括目录的 `.meta`）一起提交、移动、重命名或删除。优先在 Unity 内操作；禁止批量删除 `.meta` 来“整理”项目。
- 使用 Visible Meta Files 和 Force Text，让可文本化的场景、预制体能审阅差异。并非所有 `.asset` 都是文本；大型二进制 `.asset` 按实际路径单独配置 LFS。
- 缓存、临时文件、本地编辑器状态和构建输出由 `.gitignore` 排除。不要忽略整个 `Packages/`、`ProjectSettings/` 或所有 `.dll`，这些可能是必需依赖。
- 场景和预制体发生冲突时仍需检查资源引用。当前没有启用需每台机器单独安装配置的 UnityYAMLMerge 驱动。

接入现有工程的顺序见 [game/README.md](../game/README.md)。

## 2. Git LFS 已配置什么

根目录 `.gitattributes` 将常见模型源文件、模型导出物、分层贴图、图像、音频、视频映射为 LFS。Git 历史保存指针，实际内容上传到 LFS 存储；文本脚本、说明和 `.meta` 仍走普通 Git。

**规则已经提交不等于你的电脑已安装 LFS，也不等于已有历史中的大文件已完成迁移。** 首次添加资源前执行 `git lfs install`。不要用网页上传或 Contents API 代替大文件的 LFS 上传流程。

下列命令在仓库根目录执行，适用于 Git Bash。示例假定你已把真实文件放在 `art/source/models/Building01.blend`；当前仓库不含该模型。

```bash
git switch -c feat/import-building01
git lfs install --local
git check-attr filter -- art/source/models/Building01.blend
# 预期：filter: lfs
git add .gitattributes art/source/models/Building01.blend
git lfs status
git lfs ls-files
git show :art/source/models/Building01.blend
# 暂存的内容应是以 version https://git-lfs.github.com/spec/v1 开头的指针。
git diff --cached --stat
git status --short
git commit -m "art: import Building01 source model"
git push -u origin feat/import-building01
```

如还接入了 Unity 资源，将实际资源路径和配套 `.meta` 一并暂存。确认 LFS 列表包含资源后再提交。上述操作完成后可通过 PR 将资源合入 `main`。

克隆到另一台电脑后：

```bash
git lfs install --local
git lfs pull
git lfs fsck
```

如果模型文件变成几行 `version / oid / size` 文本，说明只取得了指针。先修复 LFS 下载和权限，再用 Unity 或建模软件打开；不要保存覆盖指针文件。

## 3. 新格式、超大文件与费用

- `.gitignore` 按路径工作，**不会按文件大小自动拦截上传**。新格式应先用 `git check-attr filter -- 路径` 核对。
- 新遇到的大型二进制文件可按精确路径添加规则，例如 `git lfs track "game/UnknownCity/Assets/_Project/Terrain/Building01Terrain.asset"`，再将更新后的 `.gitattributes` 和文件一起提交。不要把全部 `.asset` 放入 LFS。
- GitHub 普通 Git 对超过 50 MiB 的文件发出警告，对超过 100 MiB 的文件拒绝推送。此限制不是建议等到 100 MiB 才使用 LFS；频繁变化的模型和贴图应从第一次提交起使用 LFS。
- LFS 自身仍有单文件限制、存储与下载额度，按账号方案及当时政策计算。修改并重新推送大文件会增加存储用量；本次没有上传二进制资源，也没有启用任何付费预算。
- 超大扫描件、原始视频、渲染缓存和整包备份优先放外部存储；构建包按需要发布为 GitHub Release 附件或其他分发包。用 [资源登记表](ASSET_REGISTER.md) 记录固定版本、获取方式与 SHA-256。

## 4. 已经误提交时

给 `.gitignore` 加规则不会清除已有跟踪或历史；给 `.gitattributes` 加规则也不会追溯转换历史文件。

尚未提交到共享历史的文件，可以先取消暂存、设置正确规则，再重新暂存检查。已经推送的大文件要先分析影响、备份并协调，再考虑 LFS 迁移或历史清理；不要直接强制推送覆盖他人的提交。

若误推送的是凭据，先撤销或轮换凭据。删除最新一版文件不能使历史中的凭据失效。

## 5. 每次资源交接前

- `git status` / `git diff --cached` 中只包含预期内容，没有缓存、私密文件和无许可素材。
- 模型在 `git lfs ls-files` 中出现，Unity 文件与 `.meta` 成对，依赖不指向个人磁盘的绝对路径。
- 推送同时上传了 Git 提交和 LFS 对象；在干净克隆或另一台机器上验证关键场景能打开。
- 二进制资源避免并行覆盖；源文件和外部资源另有可恢复的备份。LFS 配额或权限异常时，不能只靠仓库指针恢复原文件。

## 官方资料

规则核对日期：2026-10-09。下面是版本管理参考，不代表本工程已选定这些 Unity 版本。

- [Unity：工程纳入外部版本管理](https://docs.unity3d.com/2020.1/Documentation/Manual/ExternalVersionControlSystemSupport.html)
- [Unity：资源元数据与 GUID](https://docs.unity3d.com/6000.1/Documentation/Manual/AssetMetadata.html)
- [Unity：Editor 的 Asset Serialization 设置](https://docs.unity3d.com/6000.0/Documentation/Manual/class-EditorManager.html)
- [GitHub：配置 Git LFS](https://docs.github.com/en/repositories/working-with-files/managing-large-files/configuring-git-large-file-storage)
- [GitHub：普通 Git 大文件限制](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github)
- [GitHub：LFS 单文件与方案限制](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage)
- [GitHub：LFS 计费规则](https://docs.github.com/en/billing/concepts/product-billing/git-lfs)
