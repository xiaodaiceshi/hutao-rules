# Hutao Rules

适用于 Mihomo / Stash 客户端的代理规则配置。参考自 [Lanlan13-14/Rules](https://github.com/Lanlan13-14/Rules) 并适配 hutao-rules。

## 配置文件

| 文件 | 说明 | 对应参考仓库文件 |
|---|---|---|
| `hutao-full.yaml` | 完整版（含广告拦截） | `configfull.yaml` |
| `hutao-full-noad.yaml` | 完整版（无广告拦截） | `configfull_NoAd.yaml` |
| `hutao-lite.yaml` | 精简版（含广告拦截） | `configfull_lite.yaml` |
| `hutao-beta.yaml` | 测试版（精简服务分组） | `configfull_beta.yaml` |
| `hutao-custom.ini` | 自定义配置 | `custom-mini.ini` |
| `pcdn.list` | PCDN 列表 | `pcdn.list` |

## 仓库结构

```
hutao-rules/
├── .github/workflows/        CI/CD 自动化
├── src/
│   ├── rules/                规则集
│   │   ├── domain/           域名规则
│   │   ├── game/             游戏规则
│   │   └── ip/               IP 规则
│   └── rewrite/              复写规则
├── docs/                     文档
├── icon/                     图标资源（指向本仓库）
├── input/                    上游参考规则
├── notes/                    项目笔记
├── hutao-full.yaml           ← 完整配置
├── hutao-full-noad.yaml      ← 无广告配置
├── hutao-lite.yaml           ← 精简配置
├── hutao-beta.yaml           ← 测试配置
├── README.md
├── pcdn.list
└── LICENSE
```

## 使用方式

1. 下载对应版本的 `.yaml` 配置文件
2. 在文件中填入你的订阅链接（搜索 `订阅链接1`）
3. 导入到 Mihomo / Stash 客户端

## 内部引用

所有自定义规则集和图标均引用本仓库：
- `https://raw.githubusercontent.com/xiaodaiceshi/hutao-rules/refs/heads/main/src/rules/...`
- `https://raw.githubusercontent.com/xiaodaiceshi/hutao-rules/refs/heads/main/icon/...`

## 参考

- [Lanlan13-14/Rules](https://github.com/Lanlan13-14/Rules) - 本项目的参考上游
- [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)
- [Aethersailor](https://github.com/aethersailor/Custom_OpenClash_Rules)
- [MetaCubeX](https://github.com/MetaCubeX/meta-rules-dat)

## License

MIT