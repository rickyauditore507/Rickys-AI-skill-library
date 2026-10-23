# curl 直连一手来源核实配方

> 场景：内置 `web_search`/`web_extract` 不可用（后端超时、SearXNG 报错、返回垃圾），或需要可引用的一手事实（许可证、价格/免费额度、版本号、官方政策、模型卡规格、发行说明）。

## 通用技巧

```bash
UA='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
# 先做连通性矩阵，知道哪条路通再往下走
for u in https://api.github.com https://raw.githubusercontent.com https://huggingface.co https://www.bing.com; do
  echo "$u -> $(curl -s -m 12 -o /dev/null -w '%{http_code}' -A "$UA" "$u")"
done
```

- 每个请求都带 `-m <秒>` 超时；用 `-w "%{http_code} %{size_download}"` 自检返回码/体积。
- HTML 站点带 UA；跟随重定向用 `-sL`（不加 `-L` 常见只拿到 302）。
- 抓 JSON 字段用 `grep -oE`（不依赖 jq）。
- URL 含 `&` 必须整体加引号；git-bash 支持中文路径。

## 配方

### 1) GitHub：组织仓库清单 / 单仓库许可证

```bash
curl -s -m 20 -A "$UA" "https://api.github.com/orgs/<ORG>/repos?per_page=100&sort=pushed" \
  | grep -o '"full_name": *"[^"]*"' | head -50          # 仓库清单（最近 push 在前）
curl -s -m 15 -A "$UA" "https://api.github.com/repos/<ORG>/<REPO>" \
  | grep -o '"license":[^}]*}' | head -1                # 单仓库许可证
```

坑：无鉴权约 60 次/小时；`"license":null` 只说明仓库根目录没有标准许可证文件——**不等于没有许可证**（很多项目把许可证随权重/产物发布，写在 README 或 HF/ModelScope 模型卡），要去补证。

### 2) 原始文件（README、LICENSE、任意文本）

```bash
curl -s -m 15 -A "$UA" https://raw.githubusercontent.com/<ORG>/<REPO>/main/README.md \
  | grep -inE 'license|apache' | head
```

### 3) HuggingFace（国内直连常不通 → hf-mirror.com 同路径镜像）

```bash
code=$(curl -s -m 12 -o /tmp/hf.json -w "%{http_code}" "https://huggingface.co/api/models?author=<ORG>&sort=lastModified&direction=-1&limit=100")
[ "$code" != "200" ] && curl -s -m 20 -o /tmp/hf.json "https://hf-mirror.com/api/models?author=<ORG>&sort=lastModified&direction=-1&limit=100"
grep -oE '"id": *"<ORG>/[^"]*"' /tmp/hf.json | head -20                      # 模型 ID
grep -oE 'license:[a-zA-Z0-9.-]+' /tmp/hf.json | sort | uniq -c | sort -rn    # 许可证分布
curl -s -m 15 https://hf-mirror.com/<ORG>/<MODEL>/raw/main/README.md | grep -inE 'license' | head   # 单卡细节（license_name 等）
```

### 4) 厂商文档页（JS 站也能抠出正文，阿里云帮助页可抓）

```bash
curl -sL -m 25 -A "$UA" "https://help.aliyun.com/zh/..." -o /tmp/p.html -w "http=%{http_code} size=%{size_download}\n"
sed 's/<[^>]*>/ /g' /tmp/p.html | grep -oE '.{0,30}(免费|额度|价格|quota).{0,80}' | head -20
```

### 5) 搜索引擎 HTML 兜底（只当线索，别当结论）

```bash
curl -sL -m 25 -A "$UA" "https://www.bing.com/search?q=<urlencoded>&mkt=zh-CN" -o /tmp/b.html -w "%{http_code} %{size_download}\n"
sed 's/<\/h2>/\n/g' /tmp/b.html | grep -oE '<h2[^>]*>.*' | sed 's/<[^>]*>//g' | head -12
```

结果常混入无关条目——拿到线索后回到一手源确认。

## 交付格式

结论按「事实 → 来源(URL) + 原文片段」成对给出；末尾单列：

1. 用了哪些源；
2. 哪些结论**没**验证到。

不要用「应该」「大概」替代已抓到的原文。
