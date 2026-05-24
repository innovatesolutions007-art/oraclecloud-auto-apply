# OCI A1 Instance Auto Grab

自动抢占 Oracle Cloud 永久免费 A1 (ARM) 实例的 GitHub Actions 工作流。

## 背景

Oracle Cloud 免费套餐提供 4 OCPU / 24GB 内存的 ARM (Ampere A1) 实例，永久免费。
但热门区域（如 US East Ashburn）资源紧张，手动创建经常遇到 "Out of host capacity" 错误。
本项目通过定时任务自动重试，在资源释放时直接抢占满配实例。

## 策略

- 直接创建 4 OCPU / 24GB 满配实例，不做渐进式扩容
- 依次尝试同区域三个 Availability Domain（AD-1、AD-2、AD-3）
- 任意一个 AD 成功即停止，不会重复创建
- 已有实例运行时自动跳过，不会误建多个

## 定时调度

美东时间凌晨 12:00 - 7:00（UTC 4:00 - 11:00），每 30 分钟运行一次。
也支持手动触发（workflow_dispatch）。

## 成功案例

- 区域：US East (Ashburn)
- 账号类型：Pay As You Go（升级后抢到概率更高）
- 实例配置：VM.Standard.A1.Flex，4 OCPU，24GB 内存
- 系统镜像：Canonical Ubuntu 22.04 Minimal aarch64
- 抢到时间：注册当天（跑了1次拿到了3个VM.Standard.A1.Flex，4 OCPU，24GB, 然后我手动terminate了2个来符合免费的要求）

## 配置说明

在仓库 Settings → Secrets and variables → Actions 中添加以下 Secrets：

| Secret | 说明 |
|--------|------|
| OCI_CLI_USER | User OCID |
| OCI_CLI_TENANCY | Tenancy OCID |
| OCI_CLI_FINGERPRINT | API Key 指纹 |
| OCI_CLI_KEY_CONTENT | API 私钥内容（PEM 格式，包含 BEGIN/END 行） |
| OCI_CLI_REGION | 区域，如 us-ashburn-1 |
| OCI_COMPARTMENT_ID | Compartment OCID（无自定义则同 Tenancy OCID） |
| OCI_SUBNET_ID | 子网 OCID |
| OCI_IMAGE_ID | ARM 镜像 OCID |
| OCI_SSH_PUBLIC_KEY | SSH 公钥内容（以 ssh-rsa 开头的一整行） |

## 使用方法

1. Fork 本仓库
2. 配置上述 Secrets
3. 手动触发 workflow 测试：Actions → OCI A1 Instance Auto Grab → Run workflow
4. 确认日志正常后，定时任务会自动运行

## 成功后必做

抢到实例后立刻禁用 workflow，防止继续创建多余实例：

1. 点 Actions 标签
2. 点左侧 OCI A1 Instance Auto Grab
3. 点右上角 ... 三个点
4. 点 Disable workflow

## 注意事项

- Always Free 总额度为 4 OCPU / 24GB，只能有一个满配实例
- 升级到 Pay As You Go 后抢到概率更高，但务必抢到后立刻禁用 workflow
- 不禁用会导致继续创建新实例，超出免费额度会产生费用
