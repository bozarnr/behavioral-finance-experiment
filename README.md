# behavioral-finance-experiment
# 实验网页（10轮）+ 后台汇总（Excel/Google Sheet）

## 你得到了什么

- `index.html`：一个可直接打开/部署的网页  
  - 开始页 → 实验页（10轮）→ 结束页  
  - 实验页上方有 **2×11 表格**：第一行显示轮次 1-10；第二行记录每轮收益  
  - 每轮提供可勾选选项（单选），确认后把收益写入表格并进入下一轮  
  - 完成 10 轮后可“退出并提交/下载”，会下载 CSV 备份，并（可选）提交到后台汇总

## 1) 直接使用（不做后台）

1. 双击打开 `index.html`（或用任何静态托管发布，比如 GitHub Pages / 服务器静态目录）
2. 答题者完成 10 轮后会下载一份 `P-xxxx.csv`  
3. 你把收集到的 CSV 导入 Excel 汇总即可

> 如果你需要“后台自动汇总到一个总表”，继续看第 2 部分。

## 2) 后台自动汇总（推荐：Google Sheet → 导出 Excel）

核心思路：把每个答题者的 10 轮数据写入同一个 Google Sheet。你随时在 Google Sheet 里“文件 → 下载 → Microsoft Excel”拿到总 Excel。

### 2.1 新建 Google Sheet

1. 新建一个 Google Sheet，命名例如：`实验数据汇总`
2. 记住这个 Sheet 所属的 Google 账号（你后面要用它部署脚本）

### 2.2 创建 Apps Script（接收网页提交）

1. 在该 Sheet 里点击 **扩展程序 → Apps Script**
2. 删除默认内容，粘贴下面代码（完整可用）

```javascript
// Google Apps Script: 接收实验数据并写入 Google Sheet
// 部署为 Web App 后，网页会 POST JSON 到此地址

function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName("data");
    if (!sheet) {
      sheet = SpreadsheetApp.getActiveSpreadsheet().insertSheet("data");
      sheet.appendRow([
        "receivedAt",
        "participantId",
        "nickname",
        "startedAt",
        "completedAt",
        "round1",
        "round2",
        "round3",
        "round4",
        "round5",
        "round6",
        "round7",
        "round8",
        "round9",
        "round10",
        "userAgent",
        "timezone",
        "language",
        "referrer",
        "href"
      ]);
    }

    var body = e && e.postData && e.postData.contents ? e.postData.contents : "";
    var payload = JSON.parse(body);

    var payoffs = payload.payoffs || [];
    var meta = payload.meta || {};

    sheet.appendRow([
      new Date().toISOString(),
      payload.participantId || "",
      payload.nickname || "",
      payload.startedAt || "",
      payload.completedAt || "",
      payoffs[0] != null ? payoffs[0] : "",
      payoffs[1] != null ? payoffs[1] : "",
      payoffs[2] != null ? payoffs[2] : "",
      payoffs[3] != null ? payoffs[3] : "",
      payoffs[4] != null ? payoffs[4] : "",
      payoffs[5] != null ? payoffs[5] : "",
      payoffs[6] != null ? payoffs[6] : "",
      payoffs[7] != null ? payoffs[7] : "",
      payoffs[8] != null ? payoffs[8] : "",
      payoffs[9] != null ? payoffs[9] : "",
      meta.userAgent || "",
      meta.timezone || "",
      meta.language || "",
      meta.referrer || "",
      meta.href || ""
    ]);

    return ContentService
      .createTextOutput("OK")
      .setMimeType(ContentService.MimeType.TEXT);
  } catch (err) {
    return ContentService
      .createTextOutput("ERR: " + (err && err.message ? err.message : String(err)))
      .setMimeType(ContentService.MimeType.TEXT);
  }
}
```

### 2.3 部署为 Web App，并拿到提交链接

1. 点击右上角 **部署 → 新部署**
2. 类型选择 **Web 应用**
3. 执行身份建议选：**我**
4. 访问权限建议选：**任何人**（否则外部答题者无法提交）
5. 部署后复制 Web App URL（形如 `https://script.google.com/macros/s/.../exec`）

### 2.4 把提交链接写回网页

打开 `index.html`，找到这一行并填入你的 URL：

```js
const SUBMIT_URL = ""; // 例如：https://script.google.com/macros/s/xxxx/exec
```

保存后重新发布网页即可。

## 3) 发布给别人（分享链接）

你可以用任意“静态托管”把 `index.html` 变成一个 URL：

- GitHub Pages
- 你的服务器（Nginx/Apache）静态目录
- 任何支持静态网站的网盘/对象存储/CDN

只要能让别人打开网页即可；**后台汇总是通过 Apps Script 的 URL 完成的**。

## 4) 你拿到总 Excel

1. 打开你的 Google Sheet（所有答题者都会不断追加行）
2. 选择 **文件 → 下载 → Microsoft Excel (.xlsx)**  
   得到一份“该时间段内所有参与者 + 每轮收益”的总表

