# 修复审查发现的全部问题

请严格按顺序执行以下修复。每一步独立 commit。

## 问题 1：domain yaml 旧文件未删除（阻塞）

codex/automate 上 src/rules/domain/ 中 21 个旧 yaml 文件与重命名后的新文件并存。

需要删除以下 21 个文件：
- NetEaseMusic-domain.yaml
- Steam-domain.yaml
- Talkatone-domain.yaml
- WeChat.yaml
- ai.yaml
- applefirmware.yaml
- appletv.yaml
- direct.yaml
- emby.yaml
- fakeip-filter.yaml
- google.yaml
- googleVPN.yaml
- iptv.yaml
- proxy.yaml
- streaming_hk.yaml
- streaming_sg.yaml
- streaming_tw.yaml
- streaming_uk.yaml
- tvb.yaml
- ubi.yaml
- wz.yaml

命令：
```powershell
cd src/rules/domain/
git rm NetEaseMusic-domain.yaml Steam-domain.yaml Talkatone-domain.yaml WeChat.yaml ai.yaml applefirmware.yaml appletv.yaml direct.yaml emby.yaml fakeip-filter.yaml google.yaml googleVPN.yaml iptv.yaml proxy.yaml streaming_hk.yaml streaming_sg.yaml streaming_tw.yaml streaming_uk.yaml tvb.yaml ubi.yaml wz.yaml
```

## 问题 2：BOM 头移除（阻塞）

所有被修改的 .yaml 文件都被添加了 UTF-8 BOM 头。用 PowerShell 批量移除：

```powershell
# 遍历所有修改过的 yaml 文件
$files = @(
  "hutao-full.yaml", "hutao-full-noad.yaml", "hutao-lite.yaml",
  ".github/workflows/deploy.yml", ".github/workflows/sync-rules.yml",
  "src/rules/domain/proxy-domain.yaml", "src/rules/domain/wz-domain.yaml",
  "src/rewrite/hutao-rewrite-full.yaml", "src/rewrite/hutao-rewrite-lite.yaml",
  "src/rewrite/hutao-rewrite-noad-lite.yaml", "src/rewrite/hutao-rewrite-noad.yaml"
)
foreach ($f in $files) {
  if (Test-Path $f) {
    $content = [System.IO.File]::ReadAllBytes($f)
    if ($content[0] -eq 0xEF -and $content[1] -eq 0xBB -and $content[2] -eq 0xBF) {
      $clean = $content[3..($content.Length-1)]
      [System.IO.File]::WriteAllBytes($f, $clean)
      Write-Host "Removed BOM from $f"
    }
  }
}
```

## 问题 3：deploy.yml paths 格式错误（阻塞）

文件中 paths 的 \n 被写成了字面量。需修复为正确格式：
```yaml
paths:
  - 'hutao-*.yaml'
  - 'hutao-*.ini'
  - 'hutao-*.list'
  - 'pcdn.list'
  - 'src/rules/game/*.list'
  - 'src/rules/domain/upstream/*.list'
  - 'src/rules/**'
  - 'src/rewrite/**'
```

## 问题 4：README.md 结构图格式错误（阻塞）

结构图中 `\n` 被写成了字面量。修复仓库结构图，确保 pcdn.list 在正确位置。

## 问题 5：sync-rules.yml 批量编译循环更新（建议）

`for f in src/rules/domain/*.yaml` 编译后 mrs 文件名会变成 `xxx.yaml` → `xxx.mrs`。
由于文件名已经是 `{xxx}-domain.yaml`，编译结果会是 `{xxx}-domain.mrs`，这与 rule-providers 引用的 URL 一致。✅ 其实不需要改。

## 执行顺序

1. 删除 21 个旧 yaml 文件 → commit 1
2. 移除所有 BOM 头 → commit 2
3. 修复 deploy.yml paths → commit 3
4. 修复 README.md 结构图 → commit 4
5. 推送并告知我

注意：
- 只修问题 1-4，问题 5 确认不需要改
- 修复完成后用 grep 验证所有 .mrs 引用与新文件名一致
