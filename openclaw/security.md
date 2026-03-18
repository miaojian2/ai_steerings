---
inclusion: manual
---

# 安全模块 (src/security/)

## 概述

安全模块负责安全审计、DM 策略、工具安全检查和配置安全验证。

## 目录结构

```
src/security/
├── audit.ts               # 安全审计主入口
├── audit-*.ts             # 审计子模块
├── dm-policy-shared.ts    # DM 策略
├── dangerous-tools.ts     # 危险工具检测
├── dangerous-config-flags.ts # 危险配置标志
├── external-content.ts    # 外部内容安全
├── safe-regex.ts          # 正则表达式安全
├── skill-scanner.ts       # 技能安全扫描
├── fix.ts                 # 安全修复建议
└── *.test.ts              # 测试文件
```

## 安全审计 (audit.ts)

### 运行审计

```typescript
import { runSecurityAudit } from "./audit.js";

const report = await runSecurityAudit({
  config: cfg,
  deep: true,                    // 深度检查
  includeFilesystem: true,       // 文件系统权限检查
  includeChannelSecurity: true   // 渠道安全检查
});

// 报告结构
// {
//   ts: 1234567890,
//   summary: { critical: 0, warn: 2, info: 5 },
//   findings: [...]
// }
```

### 审计发现

```typescript
type SecurityAuditFinding = {
  checkId: string;           // 检查 ID
  severity: "info" | "warn" | "critical";
  title: string;             // 标题
  detail: string;            // 详细描述
  remediation?: string;      // 修复建议
};
```

### 检查项目

- Gateway 认证配置
- 文件系统权限
- Tailscale 暴露
- Control UI 安全
- 危险配置标志
- 渠道安全配置

## DM 策略 (dm-policy-shared.ts)

```typescript
type DmPolicy = "open" | "pairing" | "allowlist" | "closed";
```

- `open`: 接受所有 DM
- `pairing`: 需要配对码
- `allowlist`: 仅允许列表中的用户
- `closed`: 拒绝所有 DM

## 危险工具检测 (dangerous-tools.ts)

```typescript
import { DEFAULT_GATEWAY_HTTP_TOOL_DENY } from "./dangerous-tools.js";

// 默认禁止通过 HTTP 调用的工具
// ["session_spawn", "session_control", ...]
```

## 文件系统安全 (audit-fs.ts)

```typescript
import { inspectPathPermissions } from "./audit-fs.js";

const perms = await inspectPathPermissions("/path/to/check");
// {
//   ok: true,
//   worldWritable: false,
//   groupWritable: false,
//   worldReadable: true,
//   ...
// }
```

## 外部内容安全 (external-content.ts)

```typescript
import { sanitizeExternalContent } from "./external-content.js";

// 清理外部内容中的潜在危险元素
const safe = sanitizeExternalContent(untrustedContent);
```

## 正则表达式安全 (safe-regex.ts)

```typescript
import { isSafeRegex } from "./safe-regex.js";

// 检查正则表达式是否安全（防止 ReDoS）
const safe = isSafeRegex(pattern);
```

## 技能安全扫描 (skill-scanner.ts)

```typescript
import { scanSkillSecurity } from "./skill-scanner.js";

// 扫描技能目录的安全问题
const issues = await scanSkillSecurity(skillPath);
```

## 配置安全标志 (dangerous-config-flags.ts)

```typescript
import { collectEnabledInsecureOrDangerousFlags } from "./dangerous-config-flags.js";

// 收集启用的危险配置标志
const flags = collectEnabledInsecureOrDangerousFlags(cfg);
// ["dangerouslyDisableDeviceAuth", ...]
```

## 审计子模块

- `audit.runtime.ts` - 运行时审计
- `audit.deep.runtime.ts` - 深度审计
- `audit.nondeep.runtime.ts` - 非深度审计
- `audit-channel.ts` - 渠道审计
- `audit-channel.collect.runtime.ts` - 渠道审计收集
- `audit-tool-policy.ts` - 工具策略审计
- `audit-extra.ts` - 额外审计检查

## CLI 命令

```bash
# 运行安全审计
openclaw security audit

# 深度审计
openclaw security audit --deep

# 输出 JSON
openclaw security audit --json
```

## 安全最佳实践

1. 保持 Gateway 绑定为 loopback
2. 配置 Gateway 认证 token
3. 使用 `pairing` 或 `allowlist` DM 策略
4. 限制文件系统权限（600/700）
5. 避免启用危险配置标志
6. 定期运行安全审计

## 阅读建议

1. 从 `audit.ts` 了解审计流程
2. 阅读 `dm-policy-shared.ts` 理解 DM 策略
3. 查看 `dangerous-tools.ts` 了解工具安全
4. 研究 `audit-fs.ts` 了解文件系统检查
5. 查看 `SECURITY.md` 了解安全模型
