# playwright 下载 PDF 文件

传统爬虫往往局限于从 HTML/API
获取数据。在大模型时代，二进制文件也往往是爬取目标之一。

以 PDF 格式为例。chromium 点击打开 PDF 文件时，不会触发下载，而是打开内置预览。

## 长期悬置的 bug

Playwright 上游并非没有发现该问题。

2021 年的 issue
[#7822 Make PDF testing idiomatic](https://github.com/microsoft/playwright/issues/7822)
就提到了相关问题。

该问题需要上游 chromium
[提供命令行参数](https://issues.chromium.org/issues/40854398#comment24)，但是
chromium 上游决定不修复该问题。

**该 issue 认为 headless 模式没有该问题，实际上 headless 模式下 PDF preview 问题仍频繁发生。**

## 变通方案

不同相关 issue 下给出了部分变通方案，但是在实际开发中，每种方案都存在各自问题。

### 方案一：自定义 route

```ts
await page.route("**/empty.pdf", async (route) => {
  const response = await route.fetch();
  await route.fulfill({
    response,
    headers: {
      ...response.headers(),
      "Content-Disposition": "attachment",
    },
  });
});
```

**问题：** 一旦上游有奇怪的 redirection, 就会完全获取失败

### 方案二：直接请求 PDF

```py
response = await page.request.get("https://www1.hkexnews.hk/listedco/listconews/gem/2023/0209/2023020900150_c.pdf")
print(response.status)
```

**问题：**

1. 部分 PDF 链接使用 JS 动态构造，如果要逆向分析就没必要 pw 了
2. 部分网站在下载 PDF 时存在单独的复杂会话检验

### 方案三：chromium 首选项

```ts
const tmpDir = fs.mkdtempSync(`${os.tmpdir()}${path.sep}`);
fs.mkdirSync(`${tmpDir}/userdir/Default`, { recursive: true });

const defaultPreferences = {
  plugins: {
    always_open_pdf_externally: true,
  },
};

fs.writeFileSync(
  `${tmpDir}/userdir/Default/Preferences`,
  JSON.stringify(defaultPreferences),
);

const context = await chromium.launchPersistentContext(`${tmpDir}/userdir`, {
  acceptDownloads: true,
  headless: false,
  viewport: {
    width: 1440,
    height: 900,
  },
});
```

**问题：** chromium
并不完全听话，预览还是会薛定谔式地打开。不可预测对于爬虫是灾难性的。

### 方案四：download 属性

```ts
await page.$eval(
  pdfLinkSelector,
  (el) => el.setAttribute("download", "download"),
);
```

**问题：** 仅对 a 标签有用

### 方案五：换用 firefox

Firefox 可以完美解决这一问题。

**问题：** playwright 对于 firefox 和 webkit 的实现非常敷衍。如果选用，很多高级功能无法使用。

## 解决方案：自己编译

自己编译 chromium, 完全剔除 PDF 预览相关模块。

