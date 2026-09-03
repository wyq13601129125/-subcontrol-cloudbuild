# ☁️ Subcontrol 云编译 — 操作说明 (M1b-2)

用 GitHub Actions 在云端编译 **ArduSub(Subcontrol 补丁版)** 固件，产出可烧录的 `.apj`。
你**不需要上传几百 MB 源码** —— 云端自动拉上游源码 + 打补丁编译。你只需建一个小仓库 push 一次。

---

## 📦 本文件夹内容(推送仓库的全部文件)

```
云编译/  ──(push 到这个内容)──> 你的 GitHub 新仓库
├── .github/workflows/build-sub.yml   ← 编译流水线
├── .gitattributes                    ← 保证补丁 LF 换行
└── subcontrol.patch                  ← 本项目全部代码改动(4 文件)
```

- 补丁基线 = 上游 ArduPilot `cad79ca723021f326b48d916f89b4a932c5d8cf0`(与本地快照同源)
- 改动内容:新增 `AP_Motors_ControlAlloc.{h,cpp}`,修改 `AP_Motors6DOF.{h,cpp}`(CUSTOM 通用控制分配 + 推力曲线 γ)
- 编译板型 = **mRoControlZeroClassic**(唯一与自研板同芯片 STM32H743ZIT6 的官方板型),是 M2 自研 hwdef 的起点

---

## 🚀 步骤

### 第 1 步:建 GitHub 仓库(网页操作)

1. 登录 [github.com](https://github.com) → 右上角 **+** → **New repository**
2. Repository name: 填 `subcontrol-cloudbuild`(可自取)
3. **Private**(私有即可,不用公开)
4. ⚠️ **不要**勾选 “Add a README / .gitignore / license” —— 保持空仓库
5. 点 **Create repository**

### 第 2 步:本地 push(Windows Git Bash)

把下面的 `<用户名>` 换成你的 GitHub 用户名,一整段粘贴执行:

```bash
cd "D:/subcontrol-system/云编译"

git init
git config user.name  "你的GitHub用户名"
git config user.email "你的GitHub邮箱"
git add .
git commit -m "subcontrol cloud build: patch + workflow"
git branch -M main
git remote add origin https://github.com/<用户名>/subcontrol-cloudbuild.git
git push -u origin main
```

> push 弹出账号密码时:用户名填 GitHub 用户名,密码**不是登录密码**,而是 **Personal Access Token(PAT)**。
> 生成 PAT:GitHub → 头像 → Settings → Developer settings → **Personal access tokens → Tokens (classic)** →
> Generate new token → 勾选 `repo` 范围 → 复制生成的 `ghp_...` 字符串。**(只能看一次,存好)**

### 第 3 步:看编译进度

push 完成后会自动触发。打开仓库 → **Actions** 标签页 → 看到 **“Build ArduSub (Subcontrol)”** 在跑。
整个过程约 **15–40 分钟**(拉源码 + 子模块 + 编译),点击黄色/绿色圆点可看实时日志。

### 第 4 步:下载固件

编译通过(绿色 ✓)后,在本次运行页底部 **Artifacts** 区下载
`ardisub-mRoControlZeroClassic` zip → 解压得到 **`ardusub.apj`** → 用 QGC「固件」页刷入自研板。

### 后续想重跑

仓库 **Actions** 页 → 左侧选 **Build ArduSub (Subcontrol)** → 右侧 **Run workflow** 下拉 → 绿色按钮。
(改了 workflow 或想换板型后常用。)

---

## 🔧 失败排查(常见)

| 现象 | 处理 |
|---|---|
| `git apply` 报错/补丁打不上 | 告诉我日志。应急可用 `git apply --3way`(workflow 第 3 步改成 `--3way` 重 push) |
| ARM toolchain missing | 容器镜像 tag 换了 → 日志会列出 `/opt` 内容,把实际路径发我,改 workflow 第 4 步 |
| `waf configure` 卡住 | 多为子模块未拉全(网络波动),Actions 页 **Re-run** 即可 |
| 编译失败在 `AP_Motors_*` | 说明代码有编译错误 —— **这正是 M1b-2 要抓的问题**,把红字日志发我,我来修 |

---

## 📌 后续衔接

- M1b-2 通过后 → **M2**:把本 workflow 第 4 步的 `--board mRoControlZeroClassic`
  换成自研板型名(新增 hwdef),即可云编译出自研板匹配固件。
- 自研 hwdef 也可做成补丁加进 `subcontrol.patch`,一次流水线同时含代码 + 板型改动。

— 更新于 2026-09-03
