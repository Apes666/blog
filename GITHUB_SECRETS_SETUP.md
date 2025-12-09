# GitHub Secrets 快速配置指南

本指南帮助你快速配置 GitHub Actions 所需的 Secrets。

## 🎯 配置步骤

### 第一步：进入 GitHub Secrets 设置页面

1. 打开你的 GitHub 仓库
2. 点击 **Settings**（设置）
3. 在左侧菜单中找到 **Secrets and variables** → **Actions**
4. 点击 **New repository secret** 按钮

### 第二步：添加必需的 Secrets

按照下表逐个添加 Secret：

#### 1. SERVER_HOST（服务器地址）

- **Name**: `SERVER_HOST`
- **Value**: 你的服务器 IP 地址或域名
- **示例**:
  - `192.168.1.100`
  - `example.com`
  - `server.example.com`

#### 2. SERVER_USER（SSH 用户名）

- **Name**: `SERVER_USER`
- **Value**: SSH 登录用户名
- **示例**:
  - `root`
  - `ubuntu`
  - `www-data`
  - 你的自定义用户名

#### 3. SSH_PRIVATE_KEY（SSH 私钥）

- **Name**: `SSH_PRIVATE_KEY`
- **Value**: SSH 私钥的完整内容

**获取私钥内容：**

```bash
# 如果已有密钥
cat ~/.ssh/id_rsa
# 或
cat ~/.ssh/id_ed25519

# 如果需要生成新密钥
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/github_actions_key
cat ~/.ssh/github_actions_key
```

**重要提示：**
- 必须包含 `-----BEGIN ... PRIVATE KEY-----` 和 `-----END ... PRIVATE KEY-----` 行
- 复制时不要遗漏任何字符
- 私钥内容示例：

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBxxx...（中间省略）...xxxxx
-----END OPENSSH PRIVATE KEY-----
```

**配置公钥到服务器：**

```bash
# 方法 1：使用 ssh-copy-id
ssh-copy-id -i ~/.ssh/github_actions_key.pub user@your-server

# 方法 2：手动添加
# 在本地查看公钥
cat ~/.ssh/github_actions_key.pub

# 在服务器上执行
mkdir -p ~/.ssh
echo "你的公钥内容" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

#### 4. DEPLOY_PATH（部署路径）

- **Name**: `DEPLOY_PATH`
- **Value**: 服务器上的网站根目录
- **常见路径**:
  - `/var/www/html`
  - `/usr/share/nginx/html`
  - `/home/user/www`
  - `/var/www/your-site`

**确保目录存在并有正确权限：**

```bash
# 在服务器上执行
sudo mkdir -p /var/www/html
sudo chown -R your-user:your-user /var/www/html
sudo chmod -R 755 /var/www/html
```

#### 5. SERVER_PORT（可选，SSH 端口）

- **Name**: `SERVER_PORT`
- **Value**: SSH 端口号
- **默认值**: `22`
- **仅在使用非标准端口时需要配置**

## ✅ 配置检查清单

完成配置后，请确认：

- [ ] `SERVER_HOST` - 服务器地址已添加
- [ ] `SERVER_USER` - SSH 用户名已添加
- [ ] `SSH_PRIVATE_KEY` - SSH 私钥已添加（包含完整内容）
- [ ] `DEPLOY_PATH` - 部署路径已添加
- [ ] `SERVER_PORT` - （可选）如果使用非标准端口已添加
- [ ] SSH 公钥已添加到服务器的 `~/.ssh/authorized_keys`
- [ ] 服务器部署目录已创建并设置正确权限
- [ ] 可以通过 SSH 密钥成功连接服务器

## 🧪 测试 SSH 连接

在添加 Secrets 之前，先在本地测试 SSH 连接：

```bash
# 测试连接
ssh -i ~/.ssh/github_actions_key user@your-server

# 测试非标准端口
ssh -i ~/.ssh/github_actions_key -p 2222 user@your-server

# 如果连接成功，说明密钥配置正确
```

## 📋 配置示例

假设你的配置如下：

| 项目 | 值 |
|------|-----|
| 服务器 IP | `192.168.1.100` |
| SSH 用户 | `ubuntu` |
| SSH 端口 | `22`（默认） |
| 部署路径 | `/var/www/html` |

那么你需要添加的 Secrets：

```
SERVER_HOST = 192.168.1.100
SERVER_USER = ubuntu
SSH_PRIVATE_KEY = -----BEGIN OPENSSH PRIVATE KEY-----
                  （你的完整私钥内容）
                  -----END OPENSSH PRIVATE KEY-----
DEPLOY_PATH = /var/www/html
```

## 🔒 安全提示

1. **永远不要**将私钥提交到代码仓库
2. **永远不要**在公开场合分享私钥
3. **定期轮换** SSH 密钥
4. **使用专用密钥**用于 CI/CD，不要使用个人密钥
5. **限制权限**：只授予部署所需的最小权限

## 🚀 完成配置后

1. 推送代码到 `main` 分支触发自动部署
2. 或在 GitHub Actions 页面手动触发工作流
3. 查看 Actions 标签页的运行日志
4. 访问你的网站验证部署结果

## ❓ 常见问题

### Q: 如何查看已配置的 Secrets？

A: 在 Settings → Secrets and variables → Actions 页面可以看到 Secret 名称列表，但无法查看具体内容（出于安全考虑）。

### Q: 如何更新 Secret？

A: 点击 Secret 名称，然后点击 "Update secret" 按钮，输入新值并保存。

### Q: 如何删除 Secret？

A: 点击 Secret 名称，然后点击 "Remove secret" 按钮。

### Q: 私钥格式不对怎么办？

A: 确保：
- 包含完整的开始和结束标记
- 没有额外的空格或换行
- 使用正确的密钥类型（RSA 或 Ed25519）

### Q: 部署失败怎么办？

A: 查看详细的故障排查指南：[DEPLOYMENT.md](./DEPLOYMENT.md#-故障排查)

## 📚 相关文档

- [完整部署指南](./DEPLOYMENT.md)
- [GitHub Secrets 官方文档](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [SSH 密钥管理](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

---

配置完成后，你的 Hugo 站点将在每次推送到 main 分支时自动部署到服务器！🎉