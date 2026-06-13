# Hutao Rules — 规则审查与规范文档

> **审查日期：** 2026-06-11
> **审查分支：** `claude/review`
> **审查范围：** `src/rules/`、`src/rewrite/`、根目录配置文件、workflow
> **目的：** 为 Codex 侧的批量整理、格式修复、命名统一提供权威规范依据

---

## 一、现状总览

### 1.1 文件统计

| 类别 | 路径 | 文件数 |
|------|------|--------|
| 域名规则 | `src/rules/domain/` | 35 |
| IP 规则 | `src/rules/ip/` | 8 |
| 游戏规则 | `src/rules/game/` | 1 |
| 复写规则 | `src/rewrite/` | 5 |
| 主配置文件 | 根目录 `*.yaml` | 4（full, full-noad, lite, beta） |
| 自定义配置 | 根目录 `*.ini` | 1 |
| PCDN 列表 | 根目录 `*.list` | 1 |
| **合计** | | **55** |

### 1.2 输出配置格式

| 文件 | 说明 | 规则引用方式 |
|------|------|-------------|
| `hutao-full.yaml` | 完整版（含广告拦截） | RULE-SET + GEOSITE + GEOIP |
| `hutao-full-noad.yaml` | 完整版（无广告拦截） | 与 full 相同，少 banAd |
| `hutao-lite.yaml` | 精简版（含广告拦截） | GEOSITE + GEOIP（无 RULE-SET） |
| `hutao-beta.yaml` | 测试版 | 与 full 类似，精简分组 |
| `hutao-custom.ini` | 自定义配置 | 不引用规则文件 |

### 1.3 规则集分发方式

| 方式 | 说明 | 涉及文件 |
|------|------|----------|
| **自编译 mrs** | 由 `sync-rules.yml` 将 `.yaml` 转为 `.mrs`，通过 `raw.githubusercontent.com` 引用 | `ai.yaml` → `ai.mrs`, `emby.yaml` → `emby.mrs`, `google.yaml` → `google.mrs` 等 |
| **MetaCubeX 仓库** | 直接引用 `meta-rules-dat` 仓库的 `.mrs` 文件 | `geosite`/`geoip` 大类规则 |
| **自定义 .mrs** | 由 workflow 从上游拉取并编译 | `banAd.mrs`, `banAd_mini.mrs` |

---

## 二、问题清单

### 2.1 【高优先级】同规则双格式并存（8 组）

以下规则同时存在 `.list`（SSTap-Rule 格式）和 `.yaml`（payload 格式），内容不重复但格式不同：

| 规则名 | .list（SSTap 格式） | .yaml（payload 格式） |
|--------|---------------------|----------------------|
| `direct` | 2 行，`DOMAIN-SUFFIX,xxx` | 5 行，`payload:` + ` - 'xxx'` |
| `emby` | 2 行 | 3 行 |
| `fakeip-filter` | 8 行 | 11 行 |
| `googleVPN` | 3 行 | 6 行 |
| `streaming_hk` | 5 行 | 8 行 |
| `streaming_sg` | 3 行 | 8 行 |
| `streaming_tw` | 3 行 | 8 行 |
| `streaming_uk` | 4 行 | 8 行 |

**说明：** `.list` 格式用于 `sync-rules.yml` 中的上游数据源（如 Emby），`.yaml` 格式用于自编译。两者用途不同，但文件名相同易造成混淆。

**建议：** 不删除 `.list`（workflow 依赖），但重命名或统一存放位置。

### 2.2 【高优先级】命名规范不统一（domain/ 目录）

| 命名模式 | 文件数 | 示例 |
|----------|--------|------|
| `xxx.yaml`（纯小写） | 16 | `ai.yaml`, `google.yaml`, `proxy.yaml` |
| `PascalCase.yaml` | 2 | `WeChat.yaml`, `NetEaseMusic-domain.yaml` |
| `xxx-domain.yaml`（带 -domain 后缀） | 3 | `Steam-domain.yaml`, `Talkatone-domain.yaml` |
| `xxx-ip.yaml`（带 -ip 后缀） | 4（ip/ 目录） | `amazon-ip.yaml`, `steamCDN-ip.yaml` |
| `PascalCase-domain.yaml` | 1 | `NetEaseMusic-domain.yaml` |

**问题：** 同一目录内，同类规则（都是域名）使用了三种不同的命名风格：
- 纯小写：`ai.yaml`
- 带类别后缀：`Steam-domain.yaml`（但 `google.yaml` 不带）
- PascalCase：`WeChat.yaml`（但 `ai.yaml` 不用）

### 2.3 【中优先级】目录结构与分类粒度

**domain/ 目录混合了过多分类：**

| 子类别 | 文件 |
|--------|------|
| 通用代理 | `proxy.yaml`, `direct.yaml` |
| 流媒体/影视 | `emby.yaml`, `iptv.yaml`, `tvb.yaml`, `appletv.yaml` |
| 社交/通讯 | `WeChat.yaml`, `Talkatone-domain.yaml`, `googleVPN.yaml` |
| 游戏/平台 | `Steam-domain.yaml`, `ubi.yaml` |
| 地区流媒体 | `streaming_hk/sg/tw/uk.yaml` |
| 基础设施 | `ai.yaml`, `google.yaml`, `fakeip-filter.yaml` |
| 特殊用途 | `pcdn.list`, `Proxymini.list` |

**建议：** 考虑增加 `domain/category/` 子目录，或将分类信息整合到文件名。

### 2.4 【中优先级】lite 版与 full 版规则引用方式不一致

| 配置 | 规则类型 | 示例 |
|------|----------|------|
| `hutao-full.yaml` | `RULE-SET,xxx_domain,AI` | 引用自编译 `.mrs` 文件 |
| `hutao-lite.yaml` | `GEOSITE,youtube,YouTube` | 直接使用 GeoSite 分类 |

**影响：** lite 版依赖客户端内置的 GeoSite 分类，full 版依赖自编译规则集。两者路由结果可能不同。

### 2.5 【中优先级】命名大小写不一致

| 规则名 | 命名模式 | 问题 |
|--------|----------|------|
| `WeChat.yaml` | PascalCase | 与 `ai.yaml` 小写不统一 |
| `NetEaseMusic-domain.yaml` | PascalCase + -domain | 风格冲突 |
| `Talkatone-domain.yaml` | PascalCase + -domain | 同上 |
| `Steam-domain.yaml` | PascalCase + -domain | 同上 |
| `googleVPN.yaml` | camelCase | `g` 小写 `V` 大写 |
| `AS132203.yaml` | 全大写 + 数字 | IP 文件，风格与其他不同 |

### 2.6 【低优先级】测试数据残留

以下文件包含示例/测试域名，可能未清理：

- `proxy.yaml`：包含 `*.proxy.test`
- `wz.yaml`：包含 `*.wz.example.com`
- `fakeip-filter.yaml`：包含 `*.example.com`

### 2.7 【低优先级】规则集引用与源文件不对应

`hutao-full.yaml` 的 `rule-providers` 中引用了以下 `.mrs` 文件，但 `src/rules/domain/` 中不存在对应的 `.yaml` 源文件：

| 引用 | 源文件是否存在 |
|------|---------------|
| `banAd_mini.mrs` | 否（由 workflow 编译） |
| `banAd_mini.mrs` | 否（由 workflow 编译） |
| `emby.mrs` | ✅ `emby.yaml` 存在 |
| `ai.mrs` | ✅ `ai.yaml` 存在 |
| `fakeip-filter.mrs` | ✅ `fakeip-filter.yaml` 存在 |
| `google.mrs` | ✅ `google.yaml` 存在 |
| `Talkatone-domain.mrs` | ✅ `Talkatone-domain.yaml` 存在 |
| `Talkatone-ip.mrs` | ✅ `Talkatone-ip.yaml` 存在 |
| `Steam-domain.mrs` | ✅ `Steam-domain.yaml` 存在 |
| `steamCDN-ip.mrs` | ✅ `steamCDN-ip.yaml` 存在 |

**结论：** 所有引用都有对应的上游 `.yaml` 或 workflow 编译源，但 `banAd_mini.mrs` 无 `.yaml` 源（由 workflow 直接编译）。这是设计意图，非问题。

---

## 三、命名规范（Codex 执行标准）

### 3.1 域名规则文件命名

**规则：** `{服务名}-{分类}.{扩展名}`

| 字段 | 规范 | 示例 |
|------|------|------|
| `服务名` | 全小写，无空格 | `netease`, `youtube` |
| `分类` | 固定为 `domain`（域名规则统一标识） | `netease-domain` |
| `扩展名` | `.yaml`（payload 格式） | `netease-domain.yaml` |

**最终格式：`{服务名}-domain.yaml`**

### 3.2 IP 规则文件命名

**规则：** `{服务名}-ip.yaml`

| 示例（当前） | 示例（统一后） |
|------|------|
| `AS132203.yaml` | `AS132203-ip.yaml` |
| `AS24424.yaml` | `AS24424-ip.yaml` |
| `amazon-ip.yaml` | `amazon-ip.yaml`（不变） |
| `google-cn-ip.yaml` | `google-cn-ip.yaml`（不变） |

### 3.3 大小写规范

**全部小写，连字符分隔。** 例外：IP 地址中的大写字母（`AS132203`）保持不变。

| 当前 | 统一后 |
|------|--------|
| `WeChat.yaml` | `wechat-domain.yaml` |
| `NetEaseMusic-domain.yaml` | `netease-music-domain.yaml` |
| `Steam-domain.yaml` | `steam-domain.yaml` |
| `Talkatone-domain.yaml` | `talkatone-domain.yaml` |
| `googleVPN.yaml` | `google-vpn.yaml` |
| `appletv.yaml` | `apple-tv.yaml` |

### 3.4 游戏/PCDN 文件命名

| 当前 | 统一后 | 说明 |
|------|--------|------|
| `hutao_game.list` | `games.list` | 去掉 `hutao_` 前缀，复数 |
| `hutao-pcdn.list`（根目录） | `pcdn.list` | 去掉 `hutao_` 前缀 |

### 3.5 配置文件命名

| 当前 | 说明 | 建议 |
|------|------|------|
| `hutao-full.yaml` | 完整版含广告 | ✅ 不变 |
| `hutao-full-noad.yaml` | 完整版无广告 | ✅ 不变 |
| `hutao-lite.yaml` | 精简版含广告 | ✅ 不变 |
| `hutao-beta.yaml` | 测试版 | ✅ 不变 |
| `hutao-custom.ini` | 自定义 | ✅ 不变 |

### 3.6 复写规则文件命名

| 当前 | 建议 |
|------|------|
| `hutao-rewrite-full.yaml` | 不变 |
| `hutao-rewrite-noad.yaml` | 不变 |
| `hutao-rewrite-lite.yaml` | 不变 |
| `hutao-rewrite-noad-lite.yaml` | 不变 |
| `hutao.yaml` | ❌ 与根目录 `hutao-full.yaml` 混淆 → 改为 `rewrite-base.yaml` |

---

## 四、统一后的 domain/ 目录结构

```
src/rules/domain/
├── ai-domain.yaml
├── alibaba-domain.yaml
├── aliyun-domain.yaml
├── apple-firmware-domain.yaml
├── apple-tv-domain.yaml
├── bilibili-ip.yaml
├── banAd-domain.yaml              ← workflow 生成
├── banAd-mini-domain.yaml         ← workflow 生成
├── direct-domain.yaml
├── emby-domain.yaml
├── fakeip-filter-domain.yaml
├── google-domain.yaml
├── google-vpn-domain.yaml
├── google-cn-ip.yaml              ← IP 格式
├── netease-music-domain.yaml
├── proxy-domain.yaml
├── pcdn.list                      ← 保持 .list
├── private-domain.yaml
├── steam-domain.yaml
├── steamcdn-ip.yaml
├── talkatone-domain.yaml
├── talkatone-ip.yaml
├── tvb-domain.yaml
├── ubi-domain.yaml
├── wechat-domain.yaml
├── xiaomi-domain.yaml
├── streaming-hk.yaml              ← 地区流媒体，保留
├── streaming-sg.yaml
├── streaming-tw.yaml
├── streaming-uk.yaml
├── IPTV.yaml                      ← 保持（特殊用途）
├── *.list                           ← SSTap-Rule 上游源文件（保留）
```

> **注意：** `.list` 文件是 SSTap-Rule 格式的 upstream 数据源，`sync-rules.yml` 的 Emby 编译步骤依赖它们。不应删除，但建议移入 `src/rules/domain/upstream/` 子目录避免混淆。

---

## 五、Codex 侧执行清单

### Phase 1：命名统一

1. 将 `src/rules/domain/` 下所有 `.yaml` 文件重命名为 `{xxx}-domain.yaml` 格式
2. 将 `src/rules/ip/` 下所有 `.yaml` 文件重命名为 `{xxx}-ip.yaml` 格式
3. 统一全部小写（IP 地址中的大写字母除外）
4. 修复 `hutao.yaml` → `rewrite-base.yaml`

### Phase 2：格式统一

5. 将所有 `.list`（SSTap-Rule 格式）移入 `src/rules/domain/upstream/`
6. 根目录 `hutao-pcdn.list` → `pcdn.list`（去掉 `hutao_` 前缀）
7. `src/rules/game/hutao_game.list` → `games.list`

### Phase 3：关联更新

8. 更新 `hutao-full.yaml` 中所有 `rule-providers` 的 URL 引用
9. 更新 `hutao-lite.yaml` 中 `fake-ip-filter` 引用的规则集名称
10. 更新 `sync-rules.yml` 中 `.list` 文件路径引用
11. 更新 `deploy.yml` 和 `Rewrite.yml` 的 paths 触发条件
12. 更新 README.md 中的文件清单

### Phase 4：清理

13. 移除 `proxy.yaml`、`wz.yaml` 中的 `.example.com` 测试数据
14. 统一 `.gitignore`：加入 `src/rules/domain/upstream/`

---

## 六、注意事项

1. **不可删除 `.list` 文件** — `sync-rules.yml` 第 62 行引用 `emby.list`，删除会导致 workflow 失败
2. **不可修改 workflow** — 除非同时更新所有相关引用
3. **URL 路径变更** — `raw.githubusercontent.com` 引用的路径必须同步更新
4. **先更新引用再移动文件** — 避免中间状态导致 workflow 触发失败
5. **`.mrs` 是编译产物** — 不应手动修改，由 workflow 自动生成
