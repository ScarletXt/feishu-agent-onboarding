# 配置与授权参考

以下语法基于 lark-cli 1.0.95 的帮助与接入实践整理。运行前重新检查当前版本帮助；大写占位符必须替换，不要原样执行。

## 1. 检查

```sh
lark-cli --version
lark-cli config init --help
lark-cli auth login --help
lark-cli auth status --profile work-agent --json --verify
```

若 profile 尚不存在，状态检查失败是预期现象；先配置。

## 2. 复用已有应用

`--name` 指定独立配置名称；`--new` 用于创建新应用，不适合直接接入已经发布的应用。

macOS 可通过剪贴板向标准输入传递密钥：

```sh
pbpaste | lark-cli config init --app-id APP_ID_PLACEHOLDER --app-secret-stdin --brand feishu --name work-agent
```

操作顺序：先将命令放入终端但不执行，再去平台复制 App Secret，返回终端按回车。若再次复制命令，会覆盖剪贴板中的密钥。

其他系统使用经过确认的安全输入方式，将密钥传入标准输入；不要把密钥写在命令参数、环境示例文件或 shell 历史里。执行者不要回显剪贴板或完整配置。

## 3. 用户授权

只选实际需要的模块；下面仅为日历和文档的例子：

```sh
lark-cli auth login --profile work-agent --domain calendar --domain docs --no-wait --json
```

从结果中取出验证链接交给用户，必要时生成本地二维码：

```sh
lark-cli auth qrcode "VERIFICATION_URL_PLACEHOLDER" --output auth-qr.png
```

用户完成授权后，继续执行：

```sh
lark-cli auth login --profile work-agent --device-code DEVICE_CODE_PLACEHOLDER
lark-cli auth status --profile work-agent --json --verify
```

检查需要的身份是否 ready、验证是否通过以及令牌状态；字段以实际版本为准。二维码和设备码属于临时授权材料，不能作为项目附件发布。

## 4. 最小只读检查

先查帮助，再运行：

```sh
lark-cli calendar +agenda --help
lark-cli drive +search --help
lark-cli calendar +agenda --profile work-agent --as user
lark-cli drive +search --profile work-agent --as user --query "" --page-size 3 --sort edit_time
```

文档搜索只验证搜索接口，不验证正文读取、编辑或多维表格操作。不要公开返回的真实标题和链接。返回包装可能使用 `ok: true`，不应机械地要求所有命令都有 `code: 0`。

## 5. 定位失败

| 现象 | 检查顺序 |
| --- | --- |
| 应用后台无权访问 | 登录账号、目标组织、应用协作者权限 |
| 已发布但 API 权限不足 | 发布版本、调用身份、用户授权范围、资源权限 |
| 用户未就绪但机器人就绪 | 完成用户授权，不用机器人身份替代 |
| 设备码过期 | 以相同范围重新授权，不扩大权限 |
| 钥匙串读取失败 | 沙箱与运行用户权限，不先删除已有凭证 |
| 妙记私有资源无法读取 | 对应接口支持的身份及资源授权，不改成公开文件 |
| 命令要求额外确认 | 展示具体动作与对象，遵守确认机制 |
