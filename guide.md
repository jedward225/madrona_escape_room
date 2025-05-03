# Madrona Escape Room Viewer 运行指南 (Windows + VcXsrv + PuTTY)

本文档记录了在 Windows 系统上，通过 VcXsrv 和 PuTTY 成功运行 Madrona Escape Room 图形查看器 (`./viewer`) 的步骤。

## 先决条件

1.  **Windows 系统**
2.  **已安装 VcXsrv X Server**：
    *   可从 [SourceForge](https://sourceforge.net/projects/vcxsrv/) 下载。
    *   安装后需要通过 XLaunch 配置并启动。
3.  **已安装 PuTTY**：
    *   可从 [官方网站](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) 下载 `putty.exe` 和 `puttygen.exe`。
4.  **SSH 私钥文件**：标准的 OpenSSH 格式私钥文件（例如 `id_rsa`），用于连接远程服务器。
5.  **已完成 Madrona Escape Room 项目的编译**：确保远程服务器上的 `/workspace/madrona_escape_room/build` 目录下存在 `./viewer` 可执行文件。
6.  **已安装必要的远程依赖**：在远程服务器上已安装 `xauth`, `libgl1-mesa-glx`, `libglfw3`, `libvulkan1`, `mesa-vulkan-drivers`, `vulkan-tools` 等。
7.  **已创建 `madrona` Conda 环境**：并且该环境已包含项目运行所需的 Python 依赖（如 PyTorch 等）。

## 步骤

### 步骤 1：准备 PuTTY 使用的私钥 (.ppk)

PuTTY 需要使用其特定的 `.ppk` 格式私钥。如果您的私钥是 OpenSSH 格式（如 `id_rsa`），需要先进行转换：

1.  运行 `puttygen.exe`。
2.  点击 "Load" 按钮。
3.  在文件选择器中，将文件类型改为 "All Files (*.*)"。
4.  选择您的 OpenSSH **私钥**文件（例如 `id_rsa`，**不是** `id_rsa.pub`）。
5.  加载成功后，（可选但推荐）在 "Key passphrase" 处为新密钥设置密码。
6.  点击 "Save private key" 按钮，将文件保存为 `.ppk` 格式（例如 `id_rsa.ppk`）。

### 步骤 2：启动并配置 VcXsrv

1.  运行 XLaunch 程序 (VcXsrv 的配置向导)。
2.  选择显示设置："Multiple windows"。
3.  选择客户端启动："Start no client"。
4.  在 "Extra Settings" 页面，**务必勾选 "Disable access control"**。
5.  点击 "Finish"。VcXsrv 会在后台运行，并在系统托盘（屏幕右下角）显示一个图标。**确保这个图标存在**。

### 步骤 3：配置 PuTTY 会话

1.  运行 `putty.exe`。
2.  **Session 配置**:
    *   Host Name: 填入您的服务器地址 (例如 `ssh9.vast.ai`)
    *   Port: 填入端口号 (例如 `34801`)
    *   Connection type: SSH
3.  **私钥配置**:
    *   导航到左侧 "Connection" -> "SSH" -> "Auth" -> "Credentials"。
    *   点击 "Browse..." 按钮，选择您在**步骤 1** 中生成的 `.ppk` 私钥文件。
4.  **X11 转发配置**:
    *   导航到左侧 "Connection" -> "SSH" -> "X11"。
    *   勾选 "**Enable X11 forwarding**"。
    *   在 "X display location" 中输入 `localhost:0`。
5.  **保存会话 (可选)**:
    *   返回 "Session" 类别。
    *   在 "Saved Sessions" 中输入名称，点击 "Save"。

### 步骤 4：连接服务器并运行 Viewer

1.  在 PuTTY 中，点击 "Open" 按钮连接服务器。
2.  输入用户名 (例如 `root`) 和 `.ppk` 文件的密码（如果您设置了）。
3.  **登录成功后，在 PuTTY 终端中执行以下命令：**
    *   **激活 Conda 环境**:
        ```bash
        conda activate madrona
        ```
        *(确保提示符变为 `(madrona)` 开头)*
    *   **切换到构建目录**:
        ```bash
        cd /workspace/madrona_escape_room/build
        ```
    *   **运行 Viewer**:
        ```bash
        ./viewer
        ```

4.  如果一切顺利，Madrona Escape Room 的图形界面窗口应该会出现在您的 Windows 桌面上。

## 常见问题

*   **`echo $DISPLAY` 为空**:
    *   检查 VcXsrv 是否已启动并禁用了访问控制。
    *   检查 PuTTY 的 X11 转发设置是否正确勾选并填写了 `localhost:0`。
    *   确保远程服务器上的 `sshd_config` 允许 X11 转发 (`X11Forwarding yes`)。
    *   确保远程服务器已安装 `xauth`。
*   **PuTTY 报 "No supported authentication methods"**: 确保您在 PuTTY Auth/Credentials 设置中选择了正确的 `.ppk` 私钥文件（不是 `id_rsa` 或 `id_rsa.pub`）。
*   **`./viewer` 报 "Failed to initialize GLFW" 或段错误**:
    *   确保**已激活正确的 Conda 环境** (`conda activate madrona`)。
    *   确保远程服务器已安装必要的运行时库 (`libgl1-mesa-glx`, `libglfw3`, `libvulkan1`, `mesa-vulkan-drivers`)。
    *   尝试运行 `vulkaninfo` 检查 Vulkan 环境。 