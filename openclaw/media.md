---
inclusion: manual
---

# 媒体处理 (src/media/)

## 概述

媒体处理模块负责处理所有媒体文件的存储、获取、转换和 MIME 类型检测。

## 目录结构

```
src/media/
├── store.ts           # 媒体存储核心
├── fetch.ts           # 远程媒体获取
├── mime.ts            # MIME 类型检测
├── audio.ts           # 音频处理
├── base64.ts          # Base64 编解码
├── ffmpeg-exec.ts     # FFmpeg 执行封装
├── image-ops.ts       # 图像操作
├── pdf-extract.ts     # PDF 文本提取
├── server.ts          # 媒体服务器
├── temp-files.ts      # 临时文件管理
└── *.test.ts          # 测试文件
```

## 核心功能

### 媒体存储 (store.ts)

```typescript
import { saveMediaSource, saveMediaBuffer, getMediaDir } from "./store.js";

// 从 URL 或本地路径保存媒体
const saved = await saveMediaSource("https://example.com/image.png");
// 返回: { id, path, size, contentType }

// 从 Buffer 保存媒体
const saved = await saveMediaBuffer(buffer, "image/png", "inbound");

// 获取媒体目录
const mediaDir = getMediaDir();  // ~/.openclaw/media/
```

### 媒体获取 (fetch.ts)

```typescript
import { fetchMedia } from "./fetch.js";

// 获取远程媒体
const result = await fetchMedia(url, {
  maxBytes: 5 * 1024 * 1024,
  headers: { Authorization: "Bearer ..." }
});
```

### MIME 类型检测 (mime.ts)

```typescript
import { detectMime, extensionForMime } from "./mime.js";

// 检测 MIME 类型（结合 magic bytes 和文件扩展名）
const mime = await detectMime({
  buffer: fileBuffer,
  headerMime: "application/octet-stream",
  filePath: "image.png"
});

// 获取扩展名
const ext = extensionForMime("image/png");  // ".png"
```

### 音频处理 (audio.ts)

```typescript
import { convertAudioToOpus, getAudioDuration } from "./audio.js";

// 转换音频格式
const opusBuffer = await convertAudioToOpus(inputBuffer);

// 获取音频时长
const duration = await getAudioDuration(audioPath);
```

### FFmpeg 封装 (ffmpeg-exec.ts)

```typescript
import { execFfmpeg, isFfmpegAvailable } from "./ffmpeg-exec.js";

// 检查 FFmpeg 可用性
const available = await isFfmpegAvailable();

// 执行 FFmpeg 命令
await execFfmpeg(["-i", input, "-c:a", "libopus", output]);
```

## 安全考虑

### 路径安全

- 使用 `inbound-path-policy.ts` 验证入站路径
- 防止路径遍历攻击
- 限制访问工作区外的文件

### 大小限制

```typescript
const MEDIA_MAX_BYTES = 5 * 1024 * 1024;  // 5MB 默认限制
```

### SSRF 防护

- 远程获取使用 `resolvePinnedHostname` 防止 SSRF
- 只允许 HTTP/HTTPS 协议
- 限制重定向次数

## 媒体服务器 (server.ts)

为 Agent 提供本地媒体访问：

```typescript
import { createMediaServer } from "./server.js";

const server = createMediaServer({
  port: 8080,
  mediaDir: getMediaDir()
});
```

## 临时文件管理

```typescript
import { createTempMediaFile, cleanupTempFiles } from "./temp-files.js";

// 创建临时文件
const tempPath = await createTempMediaFile(buffer, ".mp3");

// 清理过期临时文件
await cleanupTempFiles();
```

## 文件名处理

```typescript
import { extractOriginalFilename } from "./store.js";

// 从存储路径提取原始文件名
// "document---uuid.pdf" → "document.pdf"
const original = extractOriginalFilename(storedPath);
```

## 阅读建议

1. 从 `store.ts` 了解存储机制
2. 阅读 `mime.ts` 理解类型检测
3. 查看 `fetch.ts` 了解远程获取
4. 研究 `audio.ts` 了解音频处理
5. 查看安全相关的 `inbound-path-policy.ts`
