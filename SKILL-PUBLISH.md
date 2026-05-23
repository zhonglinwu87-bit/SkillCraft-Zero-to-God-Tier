# SKILL-PUBLISH

> 🤖 GitHub 发布助手 — 安全扫描、智能提交、一键推送。不泄露、不乱改、不丢东西。

---

## 一、触发

```
用户说：推送 / 上传github / 发布 / publish / push / 更新仓库
```

---

## 二、强制安全检查（不可跳过，每次推送前必须执行）

### 2.1 API Key / 密钥扫描

**在 `git add` 之前**，扫描所有待提交文件中的敏感信息：

```
高危模式（命中则阻断提交）：
  sk-.........          # OpenAI / DeepSeek / 任何 API Key
  ghp_.........         # GitHub Personal Access Token
  xox[bprs]-.........   # Slack Token
  AKIA..........        # AWS Access Key
  AIza..........        # Google API Key
  -----BEGIN.*PRIVATE KEY-----  # SSH / TLS 私钥
  eyJ.........          # JWT Token（长串 base64）

中危模式（警告后确认）：
  password: "..."       # 明文密码
  token: "..."          # 通用 token
  secret: "..."         # 通用 secret
  api_key: "..."        # 通用 api key
  .env                  # 环境变量文件
  credentials.json      # 凭证文件
```

**发现高危 → 立即停止 → 列出文件路径 + 行号 → 要求用户移除后再重试。**

### 2.2 大文件检查

```
检查: 单个文件 > 10MB → 警告（可能是 PDF/视频，不适合 GitHub）
检查: .pdf / .mp4 / .zip 等二进制文件 → 列出，确认是否提交
```

### 2.3 配置文件检查

```
检查: .obsidian/ 下的个人配置是否包含敏感信息
检查: 是否有系统临时文件（%TEMP%、node_modules/、.next/）
```

---

## 三、变更分析

### 3.1 变更概览

```
① git status --short → 列出所有变更
② 分类：
   M 修改   → 列出文件
   D 删除   → 列出文件
   ?? 新增  → 列出文件（排除 .gitignore 中的）
   R 重命名 → 列出文件
```

### 3.2 智能过滤

**默认排除**（不提交）：
```
.obsidian/     → Obsidian 个人配置
node_modules/  → 依赖
.temp/ / %TEMP%/ → 临时文件
.DS_Store      → macOS 系统文件
Thumbs.db      → Windows 系统文件
```

**如用户要求同时提交其他文件（如 知识库/日志.md）→ 先告知，确认后再加入。**

### 3.3 变更摘要

向用户展示一句话总结：
```
本次变更: M=3个修改 D=0个删除 A=2个新增
主要内容: [README更新] [新增Skill文件] [删除重复文件]
```

---

## 四、提交

### 4.1 提交格式

```
<type>: <简短中文描述>

详细内容用列表，每条一行。
- 新增: xxx
- 删除: xxx
- 更新: xxx

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

type 取值：
- `feat:` 新功能/新文件
- `fix:` 修复/恢复
- `docs:` 文档更新
- `chore:` 维护（删文件、改名、结构调整）

### 4.2 提交前确认

提交前展示完整的 commit message → 用户确认 → 再执行 `git commit`。

---

## 五、推送

### 5.1 远程检查

```
① git remote -v → 确认远程地址
② git branch --show-current → 确认当前分支
③ git status → 确认无遗漏文件
```

### 5.2 推送执行

```
git push origin <branch>
```

### 5.3 推送后验证

```
① 确认推送成功 → 输出 commit hash
② 检查 GitHub URL 是否可访问
③ 如果失败：检查 SSH 配置 / 网络 / 权限
```

---

## 六、结构保护协议

> 📚 经验教训：仓库结构一旦确定，推送更新时绝不擅自改变。

### 6.1 结构记忆

首次使用时记录当前仓库结构：
```
仓库根目录文件: README.md, VERSION, .gitignore, ...
顶级文件夹: Level0-先修课/, Level1-新手村/, ...
```

### 6.2 变更限制

```
禁止操作（除非用户明确指令）：
  ✋ git mv 整个目录
  ✋ 修改 .gitignore 中的目录排除规则
  ✋ 改变文件的层级结构
  ✋ 合并/拆分文件夹

允许操作：
  ✅ 新增文件（不影响已有结构）
  ✅ 修改文件内容
  ✅ 删除文件（用户确认后）
  ✅ 更新 VERSION、README、总纲
```

---

## 七、工作流速查

```
□ [安全] 扫描所有待提交文件 → 无 API Key / 密钥泄露
□ [安全] 检查大文件 + 二进制文件 → 确认
□ [分析] git status → 分类 M/D/A → 展示摘要
□ [过滤] 排除 .obsidian/node_modules/temp → 确认
□ [提交] 生成 commit message → 用户确认 → git commit
□ [推送] git push → 验证成功
```

---

> ⚠️ **铁律**：安全扫描不过绝不提交。结构不擅自改动。每次推送前必须过 checklist。
