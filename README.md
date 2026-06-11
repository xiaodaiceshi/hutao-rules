# Hutao Rules

适用于 Mihomo / Stash 客户端的代理规则配置。

## 配置文件

| 文件 | 说明 |
|---|---|
| `hutao-full.yaml` | 完整版配置 |
| `hutao-full-noad.yaml` | 完整版（无广告拦截） |
| `hutao-lite.yaml` | 精简版配置 |
| `hutao-beta.yaml` | 测试版配置 |
| `hutao-custom.ini` | 自定义配置 |
| `hutao-pcdn.list` | PCDN 列表 |

## 目录说明

```
.github/workflows/     CI/CD 自动化
src/
  rules/               规则集
    domain/            域名规则
    game/              游戏规则
    ip/                IP 规则
  rewrite/             复写规则
  scripts/             辅助脚本
docs/                  文档
icon/                  图标资源
input/                 上游参考规则
output/                生成的成品配置（已 gitignore）
notes/                 项目笔记
```

## 使用方式

1. 下载对应版本的 `.yaml` 配置文件
2. 导入到 Mihomo / Stash 客户端
3. 按需修改 `hutao-custom.ini`

## 参考

规则参考自 [ACL4SSR](https://github.com/ACL4SSR/ACL4SSR) 和 [Aethersailor](https://github.com/aethersailor/Custom_OpenClash_Rules)。

## License

MIT
