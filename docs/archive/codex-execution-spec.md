# Codex 协同执行规范

> **版本：** 1.0
> **创建日期：** 2026-06-11
> **依据文档：** `docs/claude-review.md`（来自 Claude 侧 `claude/review` 分支）
> **目标分支：** `codex/automate`

---

## 一、环境配置

### 1.1 工作目录

```
D:\sync\file\Codex_Workspace\projects\P001_hutao-rules
```

### 1.2 分支操作

```powershell
# 每次工作前确保基于最新 main
git fetch origin main
git reset --hard origin/main
git checkout -b codex/automate    # 首次创建，后续直接用
```

### 1.3 远端同步

```powershell
# 提交完成后推送
git push origin codex/automate

# 如有 Claude 侧更新，fetch 读取
git fetch origin claude/review
git show origin/claude/review:<路径>   # 读取规范文档
```

---

## 二、执行顺序

按照 Claude 侧规范文档中的 Phase 顺序执行：

```
Phase 1: 命名统一 → Phase 2: 格式统一 → Phase 3: 关联更新 → Phase 4: 清理
```

**每次 Phase 完成后**：提交一次，推送远端，等待确认再进入下一阶段。

---

## 三、各 Phase 详细执行清单

### Phase 1：命名统一

**目标：** 统一所有规则文件命名规范

| 步骤 | 操作 | 范围 | 是否可回滚 |
|------|------|------|-----------|
| 1.1 | `src/rules/domain/` 下所有 `.yaml` 重命名为 `{xxx}-domain.yaml` | 35 个文件 | ✅ |
| 1.2 | `src/rules/ip/` 下所有 `.yaml` 重命名为 `{xxx}-ip.yaml` | 8 个文件 | ✅ |
| 1.3 | 大小写统一为小写（IP 地址字母除外） | 同上 | ✅ |
| 1.4 | `src/rewrite/hutao.yaml` → `src/rewrite/rewrite-base.yaml` | 1 个文件 | ✅ |

**命名映射表：**

#### domain/ 文件

| 当前 | → | 新 |
|------|---|---|
| `NetEaseMusic-domain.yaml` | → | `netease-music-domain.yaml` |
| `Steam-domain.yaml` | → | `steam-domain.yaml` |
| `Talkatone-domain.yaml` | → | `talkatone-domain.yaml` |
| `WeChat.yaml` | → | `wechat-domain.yaml` |
| `ai.yaml` | → | `ai-domain.yaml` |
| `applefirmware.yaml` | → | `apple-firmware-domain.yaml` |
| `appletv.yaml` | → | `apple-tv-domain.yaml` |
| `direct.yaml` | → | `direct-domain.yaml` |
| `emby.yaml` | → | `emby-domain.yaml` |
| `fakeip-filter.yaml` | → | `fakeip-filter-domain.yaml` |
| `google.yaml` | → | `google-domain.yaml` |
| `googleVPN.yaml` | → | `google-vpn.yaml` |
| `iptv.yaml` | → | `iptv-domain.yaml` |
| `proxy.yaml` | → | `proxy-domain.yaml` |
| `streaming_hk.yaml` | → | `streaming-hk-domain.yaml` |
| `streaming_sg.yaml` | → | `streaming-sg-domain.yaml` |
| `streaming_tw.yaml` | → | `streaming-tw-domain.yaml` |
| `streaming_uk.yaml` | → | `streaming-uk-domain.yaml` |
| `tvb.yaml` | → | `tvb-domain.yaml` |
| `ubi.yaml` | → | `ubi-domain.yaml` |
| `wz.yaml` | → | `wz-domain.yaml` |
| `.list` 文件（13 个） | → | **不改动** |

#### ip/ 文件

| 当前 | → | 新 |
|------|---|---|
| `AS132203.yaml` | → | `AS132203-ip.yaml` |
| `AS24424.yaml` | → | `AS24424-ip.yaml` |
| `AS49544.yaml` | → | `AS49544-ip.yaml` |
| `Talkatone-ip.yaml` | → | `talkatone-ip.yaml` |
| `amazon-ip.yaml` | → | `amazon-ip.yaml`（不变） |
| `emby-ip.yaml` | → | `emby-ip.yaml`（不变） |
| `google-cn-ip.yaml` | → | `google-cn-ip.yaml`（不变） |
| `steamCDN-ip.yaml` | → | `steamcdn-ip.yaml` |

#### 其他

| 当前 | → | 新 |
|------|---|---|
| `src/rewrite/hutao.yaml` | → | `src/rewrite/rewrite-base.yaml` |
| `src/rules/game/hutao_game.list` | → | `src/rules/game/games.list`（Phase 2 执行） |

**约束：**
- Phase 1 **只重命名文件，不更新任何引用**。引用更新留给 Phase 3。
- 每个重命名后 `git mv`，确保 git 追踪。
- 完成后输出完整的改名对照表（old → new）。

---

### Phase 2：格式统一

| 步骤 | 操作 | 说明 |
|------|------|------|
| 2.1 | 创建 `src/rules/domain/upstream/` 目录 | |
| 2.2 | 将 13 个 `.list` 文件移入 `upstream/` | 不删除，只移动 |
| 2.3 | `hutao-pcdn.list`（根目录）→ `pcdn.list` | 去掉 `hutao_` 前缀 |
| 2.4 | `src/rules/game/hutao_game.list` → `src/rules/game/games.list` | 去掉前缀，复数 |

**约束：** Phase 2 **不更新引用**。

---

### Phase 3：关联更新

**这是最关键、最容易出错的阶段。** 必须确保所有引用同步更新。

#### 3.1 更新 `hutao-full.yaml`

搜索并更新所有 `rule-providers` 中的 `.mrs` URL 引用：
- `src/rules/domain/WeChat.mrs` → `src/rules/domain/wechat-domain.mrs`
- `src/rules/domain/Steam-domain.mrs` → `src/rules/domain/steam-domain.mrs`
- `src/rules/domain/Talkatone-domain.mrs` → `src/rules/domain/talkatone-domain.mrs`
- `src/rules/ip/Talkatone-ip.mrs` → `src/rules/ip/talkatone-ip.mrs`
- `src/rules/ip/steamCDN-ip.mrs` → `src/rules/ip/steamcdn-ip.mrs`
- 其他所有 `src/rules/domain/xxx.mrs` → `src/rules/domain/xxx-domain.mrs`

#### 3.2 更新 `hutao-lite.yaml`

- `fake-ip-filter` 中的规则集名称需要更新（如 `fakeip_filter_domain` → `fakeip-filter-domain`）

#### 3.3 更新 `sync-rules.yml`

- `.list` 文件路径变更：`src/rules/domain/emby.list` → `src/rules/domain/upstream/emby.list`
- 编译产物路径变更：`.mrs` 文件名更新
- `git add` 中的路径更新

#### 3.4 更新 `deploy.yml`

- `paths` 触发条件中涉及的 `.list` 文件路径

#### 3.5 更新 `Rewrite.yml`

- 涉及的 `.list` 和 `.mrs` 文件路径

#### 3.6 更新 `README.md`

- 仓库结构图中的文件列表
- 规则文件清单

**约束：**
- 先完成 Phase 1 和 Phase 2 的**物理文件操作并 commit**。
- Phase 3 先更新引用，再 commit。
- 每个文件修改前先用 `grep` 确认所有引用位置，避免遗漏。

---

### Phase 4：清理

| 步骤 | 操作 | 说明 |
|------|------|------|
| 4.1 | 清理 `proxy.yaml`（改名后为 `proxy-domain.yaml`）中的 `.proxy.test` 测试数据 | |
| 4.2 | 清理 `wz-domain.yaml` 中的 `.wz.example.com` 测试数据 | |
| 4.3 | 确认 `.gitignore` 正确（已在上轮审查中确认） | |

---

## 四、红线规则

以下规则**违反任何一条都会导致严重问题**：

1. **不可删除任何 `.list` 文件** — `sync-rules.yml` 引用它们
2. **不可修改 workflow 的 cron 调度** — 只改文件路径
3. **所有 URL 变更必须同步** — `raw.githubusercontent.com` 引用的路径变更会导致 404
4. **不可修改配置文件中的 proxy-groups / rules 逻辑** — 只改 rule-providers 的 URL
5. **不可改动 `.mrs` 文件** — 它们是编译产物
6. **不可改动 `hutao-full-noad.yaml` 和 `hutao-beta.yaml`** 中与 `hutao-full.yaml` 不同的部分 — 只更新共同引用的路径
7. **每次操作前先用 `grep` 验证** — 确认目标文件存在且格式正确
8. **每个 Phase 独立提交** — 避免混在一起难以回滚

---

## 五、完成检查清单

每个 Phase 完成后，执行以下验证：

### Phase 1 验证
- [ ] `ls src/rules/domain/ | grep -c "\.yaml"` 数量不变（35 个）
- [ ] `ls src/rules/ip/ | grep -c "\.yaml"` 数量不变（8 个）
- [ ] 所有 `.yaml` 文件名符合 `{xxx}-domain.yaml` 格式
- [ ] 所有 `.yaml` 文件内容未改变（`git diff --stat` 只显示 rename）

### Phase 2 验证
- [ ] `src/rules/domain/upstream/` 目录存在且包含 13 个 `.list` 文件
- [ ] 根目录不再有任何 `.list` 文件
- [ ] `src/rules/domain/` 下不再有任何 `.list` 文件

### Phase 3 验证
- [ ] `grep -r "\.mrs" hutao-full.yaml | wc -l` 数量不变
- [ ] 所有 URL 中的路径与新文件名一致
- [ ] `grep -r "emby\.list" .github/` 指向 `upstream/` 目录
- [ ] `git diff` 中所有变更与预期文件一致

### Phase 4 验证
- [ ] `proxy-domain.yaml` 中无 `.proxy.test`
- [ ] `wz-domain.yaml` 中无 `.wz.example.com`

---

## 六、回滚策略

如果任何 Phase 出问题：

```powershell
# 回滚到 Phase N 之前
git log --oneline        # 找到 Phase N 开始前的 commit hash
git reset --hard <hash>
git checkout -- .        # 恢复工作区
```

每个 Phase 独立提交，回滚到对应 commit 即可。
