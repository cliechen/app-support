# GitHub Pages 页面更新技能（cliechen/app-support）

## 用途
让其他智能体能够独立更新 `cliechen/app-support` 仓库中 `docs/` 目录下的两个 GitHub Pages 页面（`docs/index.html` 支持页、`docs/privacy.html` 隐私政策页），并提交到 `main` 分支触发 GitHub Pages 重新部署。

## 前置条件
1. **设置环境变量**：`export GITHUB_TOKEN="你的_GitHub_PAT"`，该 PAT 需具备 `repo` 权限，可读写 `cliechen/app-support` 仓库内容。
2. **网络可达**：能访问 `https://api.github.com`。
3. **目标仓库**：`https://github.com/cliechen/app-support`，分支 `main`，Pages 源为 `docs/` 目录。
4. **目标页面**：
   - 支持页：`docs/index.html` → 线上 `https://cliechen.github.io/app-support/`
   - 隐私政策页：`docs/privacy.html` → 线上 `https://cliechen.github.io/app-support/privacy`

## 工作原理
GitHub 内容 API 以 Base64 编码存储文件。更新流程：
1. `GET /repos/cliechen/app-support/contents/docs/<文件>` → 获取当前文件内容（Base64）和 `sha`。
2. 在本地对 HTML 进行字符串替换（整段替换，确保唯一匹配）。
3. 将修改后的完整 HTML 再次 Base64 编码。
4. `PUT /repos/cliechen/app-support/contents/docs/<文件>` → 携带新内容、`sha`、提交信息写入 `main` 分支。
5. GitHub Pages 自动从 `docs/` 重新构建部署（通常 1–3 分钟，CDN 可能缓存旧页面）。

## 操作步骤（供智能体执行）

### 第一步：获取当前页面内容和 sha
```bash
TOKEN="${GITHUB_TOKEN}"
REPO="cliechen/app-support"
FILE="docs/privacy.html"  # 或 docs/index.html

curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
  -H "Accept: application/vnd.github.v3+json" \
  "https://api.github.com/repos/$REPO/contents/$FILE" \
  | python3 -c "
import json,sys,base64
d=json.load(sys.stdin)
print('sha:', d['sha'])
print('content_b64:', d['content'])
"
```

### 第二步：在本地修改 HTML
用 Python 读取解码后的内容，进行 **整段替换**（查找字符串必须在文件中唯一出现），再重新 Base64 编码：

```python
import base64

# 从步骤一获取的 content_b64 解码
html = base64.b64decode(content_b64).decode()

# 示例：替换唯一片段
old = '<p>旧内容</p>'
new = '<p>新内容</p>'
assert html.count(old) == 1, f"查找字符串出现 {html.count(old)} 次，非唯一，拒绝替换"

html = html.replace(old, new)
new_b64 = base64.b64encode(html.encode()).decode()
```

### 第三步：提交修改到 main 分支
```bash
curl -s -X PUT -H "Authorization: token $TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Content-Type: application/json" \
  -d "{\"message\":\"提交信息（祈使句，一句话）\",\"content\":\"$new_b64\",\"sha\":\"$sha\"}" \
  "https://api.github.com/repos/$REPO/contents/$FILE"
```

返回 JSON 中的 `commit.sha` 即表示提交成功。

### 第四步：等待 GitHub Pages 构建
```bash
curl -s -H "Authorization: token $TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  "https://api.github.com/repos/$REPO/pages/builds/latest" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('status'))"
```
状态应为 `built`（若为 `building` 则继续等待；若为 `errored` 则需检查错误）。

### 第五步：线上验证（带缓存穿透参数）
```bash
cb=$(date +%s)
curl -s "https://cliechen.github.io/app-support/privacy?cb=$cb" | grep -c "预期关键词"
curl -s "https://cliechen.github.io/app-support/?cb=$cb" | grep -c "预期关键词"
```

## 关键约束（禁止事项）
- **不要**只修改本地不推送；必须执行 `PUT` 写入 `main` 分支。
- **不要**重排整页 HTML，只做整段字符串替换。
- **不要**修改其他 App 的段落（PopCritters、CremaShuffle 等），仅改目标页面相关段落。
- **不要**在聊天或日志中输出明文 PAT、API Key、密码。
- **不要**直接编辑 `TabCaddy.xcodeproj`；工程由 `project.yml` + XcodeGen 生成（属于 TabCaddy 本地仓库，非本技能范围）。

## 提交信息风格
使用祈使句、一句话，例如：
- `Fix TabCaddy privacy page contradictions`
- `Fix TabCaddy FAQ contradictions (history clear + offline scope)`
- `Restore missing Privacy Policy h1 title`
- `Apply TabCaddy privacy policy alignment fixes (P0 + P1)`

## 常见问题
| 现象 | 原因 | 处理 |
|------|------|------|
| Pages 状态 `errored` | 构建失败，可能 HTML 语法错误 | 检查替换后的 HTML 是否完整，重新提交 |
| 线上页面未变更 | GitHub Pages / CDN 缓存 | 等待 1–3 分钟，或用 `?cb=<时间戳>` 穿透缓存验证 |
| `sha` 不匹配 | 并发修改导致文件已变更 | 重新 `GET` 获取最新 `sha` 后再 `PUT` |
| 查找字符串出现多次 | 替换片段不唯一 | 增加更多上下文字符，确保 `count == 1` 后再替换 |

## 示例：完整 Python 更新脚本模板
保存为 `update_github_page.py`，修改 `TOKEN`、`FILE`、`OLD`、`NEW` 后运行：

```python
import base64, json, urllib.request

TOKEN = __import__("os").environ.get("GITHUB_TOKEN")
if not TOKEN:
    raise SystemExit("请先设置环境变量：export GITHUB_TOKEN=\"你的_GitHub_PAT\"")
REPO = "cliechen/app-support"
FILE = "docs/privacy.html"
OLD = "<p>要替换的旧内容</p>"
NEW = "<p>替换后的新内容</p>"
MSG = "Fix description in one imperative sentence"

def get_file():
    req = urllib.request.Request(
        f"https://api.github.com/repos/{REPO}/contents/{FILE}",
        headers={"Authorization": f"token {TOKEN}", "Accept": "application/vnd.github.v3+json"}
    )
    with urllib.request.urlopen(req) as r:
        return json.loads(r.read().decode())

def put_file(content_b64, sha):
    data = json.dumps({"message": MSG, "content": content_b64, "sha": sha}).encode()
    req = urllib.request.Request(
        f"https://api.github.com/repos/{REPO}/contents/{FILE}",
        data=data,
        headers={"Authorization": f"token {TOKEN}", "Accept": "application/vnd.github.v3+json", "Content-Type": "application/json"},
        method="PUT"
    )
    with urllib.request.urlopen(req) as r:
        return json.loads(r.read().decode())

info = get_file()
sha = info["sha"]
html = base64.b64decode(info["content"]).decode()
assert html.count(OLD) == 1, f"OLD 出现 {html.count(OLD)} 次，非唯一"
html = html.replace(OLD, NEW)
res = put_file(base64.b64encode(html.encode()).decode(), sha)
print("Commit:", res.get("commit", {}).get("sha", "OK"))
```

## 验收标准
- `PUT` 接口返回 200，`commit.sha` 不为空。
- Pages 构建状态为 `built`。
- 线上页面带 `?cb=` 穿透缓存后，能搜到新内容，搜不到旧内容。
- 页面结构完整（`<body>`、`<div class="container">`、`<h1>` 等标签成对）。
