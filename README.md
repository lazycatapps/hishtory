# hiSHtory 使用指南（Lazycat Cloud 版）

hiSHtory 是一个开源的 Shell 历史同步工具，本项目将其移植到 Lazycat Cloud 平台，帮助你在多台终端之间安全共享命令历史。本文聚焦于客户端的安装与常用操作，帮助你快速验证和排查同步情况。

## 安装方式
在每一台需要同步的机器上任选以下方式安装 hiSHtory 客户端：

- 使用官方安装脚本（推荐）：
  ```bash
  curl https://hishtory.dev/install.py | python3 -
  ```
- 或直接下载二进制文件：
  访问 https://github.com/ddworken/hishtory/releases ，根据操作系统选择对应的压缩包，解压后将可执行文件放入 `~/bin`、`/usr/local/bin` 等位置，然后执行 `hishtory install` 安装并写入 Shell Hook。

> 安装过程中会生成或提示密钥（Secret Key）。请妥善保存，并在其他设备使用 `hishtory init <密钥>` 加入同一同步组。

## 指向 Lazycat Cloud 服务
服务器部署完成后，请在每台设备的 Shell 配置文件（例如 `~/.zshrc` 或 `~/.bashrc`）中添加以下环境变量，并刷新终端：
```bash
export HISHTORY_SERVER=https://hishtory.${LAZYCAT_BOX_NAME}.heiyu.space
```
可以先使用 `echo $HISHTORY_SERVER` 确认环境变量是否生效。

## 常用命令
- 初始化并绑定现有密钥：
  ```bash
  hishtory init <密钥>
  ```
- 查看客户端状态与服务器连接情况：
  ```bash
  hishtory status
  ```
- 立刻同步本地历史到服务器：
  ```bash
  hishtory reupload
  ```
- 查询最近的历史记录：
  ```bash
  hishtory query --tail 20
  ```
- 按主机划分查看历史：
  ```bash
  hishtory query --host <主机名>
  ```
- 临时关闭或开启同步：
  ```bash
  hishtory syncing disable
  hishtory syncing enable
  ```
- 检查 Hook 是否正常注入（若发现同步停滞，先重新执行安装）：
  ```bash
  hishtory install
  ```
- 启动内置 Web UI 图形化浏览历史：
  ```bash
  hishtory start-web-ui
  ```
  执行后在浏览器打开终端中提示的地址（默认 http://127.0.0.1:8000），即可使用搜索、过滤等界面功能。

### 进阶查询示例
- 关键字与多条件组合：
  ```bash
  hishtory query "docker run" hostname:production exit_code:0
  ```
- 指定时间范围：
  ```bash
  hishtory query deploy before:2024-01-01
  hishtory query test after:2024-01-15
  ```
- 按目录或用户筛选：
  ```bash
  hishtory query pytest cwd:/home/user/my-project
  hishtory query restart user:root hostname:db-server
  ```
- 使用 AI 辅助（需配置 OpenAI Key）：
  ```bash
  export OPENAI_API_KEY='your-api-key'
  hishtory query ? "如何查找大文件"
  ```

## 个性化设置
- 查看当前配置：
  ```bash
  hishtory config-get
  ```
- 自定义展示列与时间格式：
  ```bash
  hishtory config-set displayed-columns Hostname CWD Command ExitCode
  hishtory config-set timestamp-format '2006/Jan/2 15:04'
  ```
- 添加自定义列（例如显示 Git 仓库远程地址）：
  ```bash
  hishtory config-add custom-columns git_remote \\
    '(git remote -v 2>/dev/null | grep origin 1>/dev/null ) && git remote get-url origin || true'
  hishtory config-add displayed-columns git_remote
  ```
- 调整键位与重复过滤等偏好：
  ```bash
  hishtory config-set key-bindings delete-entry "ctrl+d"
  hishtory config-set filter-duplicate-commands true
  hishtory config-set compact-mode true
  ```

## 验证同步
1. 在机器 A 执行一个易于识别的命令，例如 `echo hello-from-A`。
2. 运行 `hishtory reupload` 强制上传。
3. 在机器 B 上执行 `hishtory query 'hello-from-A'`，若能看到该命令，则说明同步成功。

## 安全与隐私提示
- hiSHtory 采用端到端加密，服务器无法解密命令内容，但仍应妥善保管密钥。
- 在命令前加空格可跳过记录（遵循原生 Shell 习惯）。
- 可使用 `hishtory syncing disable` 临时暂停记录，敏感操作后再 `hishtory syncing enable`。

## 常见问题与排查
- **历史记录停滞**：确认 Shell 配置文件中仍然 `source ~/.hishtory/config.*`，必要时重新执行 `hishtory install`。
- **密钥丢失**：从任意已配置好的机器上执行 `hishtory status` 查看当前密钥。
- **服务器无法访问**：检查网络连通性或联系 Lazycat Cloud 管理员确认服务是否运行。
- **需要离线使用**：安装时追加 `--offline` 参数，例如 `curl https://hishtory.dev/install.py | python3 - --offline`。
- **强制重新同步**：执行 `hishtory reupload` 或 `hishtory init --force <密钥>` 重建本地数据库。
- **排查日志**：实时查看客户端日志 `tail -f ~/.hishtory/hishtory.log`，了解同步失败原因。

## 致谢
- 感谢 [hiSHtory 官方项目](https://hishtory.dev) 及其社区维护者。
- 感谢 [linuxserver.io](https://www.linuxserver.io/) 提供的服务器镜像及文档。
- 感谢 Lazycat Cloud 社区的反馈与支持。

## 版权信息
本项目基于 [MIT License](LICENSE) 进行分发。
Copyright (c) 2025 Lazycat Apps.
