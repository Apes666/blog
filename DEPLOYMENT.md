# GitHub Actions 部署指南

本文档说明如何使用 GitHub Actions 将 Hugo 站点自动部署到服务器。

## 📋 前置要求

1. **GitHub 仓库**：代码已托管到 GitHub
2. **服务器**：一台运行 Linux 的服务器（推荐 Ubuntu/Debian）
3. **Web 服务器**：服务器上已安装并配置 Nginx
4. **SSH 访问**：能够通过 SSH 密钥访问服务器

## 🔑 配置 GitHub Secrets

在 GitHub 仓库中配置以下 Secrets（Settings → Secrets and variables → Actions → New repository secret）：

### 必需的 Secrets

| Secret 名称 | 说明 | 示例 |
|------------|------|------|
| `SERVER_HOST` | 服务器 IP 地址或域名 | `192.168.1.100` 或 `example.com` |
| `SERVER_USER` | SSH 登录用户名 | `root` 或 `ubuntu` |
| `SSH_PRIVATE_KEY` | SSH 私钥内容 | 见下方说明 |
| `DEPLOY_PATH` | 服务器上的部署目录 | `/var/www/html` 或 `/usr/share/nginx/html` |

### 可选的 Secrets

| Secret 名称 | 说明 | 默认值 |
|------------|------|--------|
| `SERVER_PORT` | SSH 端口 | `22` |

## 🔐 生成和配置 SSH 密钥

### 1. 在本地生成 SSH 密钥对

```bash
# 生成新的 SSH 密钥对（不要设置密码）
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/github_actions_key

# 或使用 RSA（如果服务器不支持 ed25519）
ssh-keygen -t rsa -b 4096 -C "github-actions" -f ~/.ssh/github_actions_key
```

### 2. 将公钥添加到服务器

```bash
# 方法 1：使用 ssh-copy-id（推荐）
ssh-copy-id -i ~/.ssh/github_actions_key.pub user@your-server

# 方法 2：手动添加
cat ~/.ssh/github_actions_key.pub
# 复制输出内容，然后在服务器上执行：
# echo "公钥内容" >> ~/.ssh/authorized_keys
```

### 3. 测试 SSH 连接

```bash
ssh -i ~/.ssh/github_actions_key user@your-server
```

### 4. 将私钥添加到 GitHub Secrets

```bash
# 查看私钥内容
cat ~/.ssh/github_actions_key

# 复制完整内容（包括 -----BEGIN ... 和 -----END ... 行）
# 粘贴到 GitHub Secrets 的 SSH_PRIVATE_KEY 中
```

## 🖥️ 服务器配置

### 1. 安装 Nginx

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nginx -y

# 启动 Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 2. 创建部署目录

```bash
# 创建网站目录
sudo mkdir -p /var/www/html

# 设置权限（将 user 替换为你的用户名）
sudo chown -R user:user /var/www/html
sudo chmod -R 755 /var/www/html
```

### 3. 配置 Nginx

创建或编辑 Nginx 配置文件：

```bash
sudo nano /etc/nginx/sites-available/your-site
```

添加以下配置：

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # 启用 gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/x-javascript application/xml+rss application/json;

    # 缓存静态资源
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

启用站点并重启 Nginx：

```bash
# 创建符号链接
sudo ln -s /etc/nginx/sites-available/your-site /etc/nginx/sites-enabled/

# 测试配置
sudo nginx -t

# 重启 Nginx
sudo systemctl reload nginx
```

### 4. 配置 sudo 权限（可选）

如果需要在部署时重启 Nginx，需要配置 sudo 权限：

```bash
sudo visudo
```

添加以下行（将 `user` 替换为你的用户名）：

```
user ALL=(ALL) NOPASSWD: /bin/systemctl reload nginx
user ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
```

## 🚀 部署流程

### 自动部署

当你推送代码到 `main` 分支时，GitHub Actions 会自动：

1. ✅ 检出代码
2. ✅ 安装 Hugo 和 Node.js 依赖
3. ✅ 构建静态站点
4. ✅ 通过 SCP 上传到服务器
5. ✅ 设置文件权限
6. ✅ 重启 Nginx

### 手动触发部署

1. 进入 GitHub 仓库
2. 点击 **Actions** 标签
3. 选择 **Deploy Hugo Site to Server** 工作流
4. 点击 **Run workflow** 按钮
5. 选择分支并点击 **Run workflow**

## 📊 查看部署状态

1. 进入 GitHub 仓库的 **Actions** 标签
2. 查看最新的工作流运行记录
3. 点击查看详细日志

## 🔍 故障排查

### 部署失败常见问题

#### 1. SSH 连接失败

**错误信息**：`Permission denied (publickey)`

**解决方案**：
- 确认 SSH 公钥已正确添加到服务器的 `~/.ssh/authorized_keys`
- 检查 `SSH_PRIVATE_KEY` Secret 是否包含完整的私钥内容
- 确认服务器 SSH 服务正在运行：`sudo systemctl status ssh`

#### 2. 权限问题

**错误信息**：`Permission denied` 或 `cannot create directory`

**解决方案**：
```bash
# 在服务器上设置正确的权限
sudo chown -R your-user:your-user /var/www/html
sudo chmod -R 755 /var/www/html
```

#### 3. Nginx 重启失败

**错误信息**：`sudo: no tty present and no askpass program specified`

**解决方案**：
- 按照上面的说明配置 sudo 权限
- 或者移除工作流中的 Nginx 重启步骤（手动重启）

#### 4. 构建失败

**错误信息**：`Error: Unable to locate config file or config directory`

**解决方案**：
- 确认 `hugo.toml` 文件存在于仓库根目录
- 检查 Hugo 版本是否兼容

### 查看服务器日志

```bash
# Nginx 错误日志
sudo tail -f /var/log/nginx/error.log

# Nginx 访问日志
sudo tail -f /var/log/nginx/access.log

# 系统日志
sudo journalctl -u nginx -f
```

## 🔒 安全建议

1. **使用专用的部署用户**：不要使用 root 用户
2. **限制 SSH 密钥权限**：只授予必要的权限
3. **定期更新密钥**：定期轮换 SSH 密钥
4. **启用防火墙**：只开放必要的端口（80, 443, SSH）
5. **配置 HTTPS**：使用 Let's Encrypt 配置 SSL 证书

### 配置 HTTPS（推荐）

```bash
# 安装 Certbot
sudo apt install certbot python3-certbot-nginx -y

# 获取 SSL 证书
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# 自动续期
sudo certbot renew --dry-run
```

## 📝 本地测试

在推送到 GitHub 之前，可以在本地测试构建：

```bash
# 安装依赖
npm install

# 本地开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产版本
npm run preview
```

## 🔄 更新工作流

如果需要修改部署流程，编辑 `.github/workflows/deploy.yml` 文件并推送到仓库。

## 📚 相关资源

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Nginx 官方文档](https://nginx.org/en/docs/)
- [SSH 密钥管理](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

## 💡 提示

- 首次部署可能需要几分钟时间
- 确保服务器有足够的磁盘空间
- 建议在非生产环境先测试部署流程
- 定期备份服务器数据

---

如有问题，请查看 GitHub Actions 的运行日志或联系管理员。