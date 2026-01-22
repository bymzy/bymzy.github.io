# 部署指南

## 文件结构

```
AppABC/
├── index.html          # 旧版本（保留备份）
├── pdf.html            # 新版本 PWA（推荐使用）
├── manifest.json       # PWA 配置
├── sw.js              # Service Worker（离线支持）
├── README_iPad.md     # iPad 使用指南
└── README_Deploy.md   # 本文件
```

## 快速部署

### 1. 本地测试

#### Windows PowerShell
```powershell
# Python 3.x
python -m http.server 8000

# 或使用 Node.js
npx http-server -p 8000 -o
```

访问: `http://localhost:8000/pdf.html`

#### Mac/Linux
```bash
# Python 3.x
python -m http.server 8000

# Python 2.x
python -m SimpleHTTPServer 8000

# Node.js
npx http-server -p 8000
```

### 2. 局域网访问（iPad 连接）

1. 获取电脑 IP 地址
   - Windows: `ipconfig`（查找 IPv4）
   - Mac: `ifconfig`（查找 inet）

2. iPad 在同一网络上访问
   - 输入: `http://192.168.x.x:8000/pdf.html`

### 3. 云服务器部署

#### 使用 GitHub Pages (免费)

1. 创建 GitHub 仓库
2. 上传所有文件
3. 启用 Pages 功能
4. 访问: `https://yourusername.github.io/AppABC/pdf.html`

#### 使用 Vercel (免费)

```bash
npm install -g vercel
vercel
```

#### 使用 Netlify (免费)

1. 拖放文件到 Netlify
2. 自动部署完成
3. 获得 HTTPS URL

#### 使用专业服务器

需要的配置：
- HTTPS 支持（必须，PWA 要求）
- GZIP 压缩
- 正确的 MIME 类型

Nginx 配置示例:
```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    # SSL 证书配置
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # GZIP 压缩
    gzip on;
    gzip_types text/plain text/css text/javascript;

    # 根目录
    root /var/www/pdf-reader;

    # 默认文件
    index pdf.html;

    # Service Worker 缓存策略
    location /sw.js {
        expires -1;
        add_header 'Cache-Control' 'no-store, no-cache, must-revalidate, proxy-revalidate';
    }

    # 其他资源缓存
    location ~* \.(js|css|woff2)$ {
        expires 1y;
        add_header 'Cache-Control' 'public, immutable';
    }
}
```

## iPad 特定配置

### 1. HTTPS 证书

iPad 上的 PWA 必须通过 HTTPS 访问。

自签名证书（测试）:
```bash
# 生成自签名证书
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes
```

真实证书（生产）:
- 使用 Let's Encrypt (免费)
- 使用 Cloudflare (免费 SSL)
- 使用商业 CA

### 2. Web 服务器配置

Python 测试服务器（支持 HTTPS）:
```bash
# 安装依赖
pip install http.server ssl

# 创建 server.py
python -c "
import http.server, ssl
server_address = ('0.0.0.0', 443)
httpd = http.server.HTTPSRequestHandler(('0.0.0.0', 443), ssl.SSLContext())
httpd.serve_forever()
"
```

### 3. Manifest 配置

manifest.json 中的关键字段:
```json
{
  "display": "standalone",      // 全屏独立应用
  "start_url": "/pdf.html",     // 启动路径
  "theme_color": "#2196f3",     // 主题色
  "background_color": "#ffffff", // 背景色
  "scope": "/"                  // 应用作用域
}
```

## 优化建议

### 1. 性能优化

- ✅ 启用 GZIP 压缩
- ✅ 使用 CDN
- ✅ 缓存 PDF.js 库
- ✅ 图片优化

### 2. 安全性

- ✅ HTTPS 必须
- ✅ CSP 头设置
- ✅ 定期更新依赖

### 3. 用户体验

- ✅ 使用高质量图标
- ✅ 正确的屏幕截图
- ✅ 响应式设计
- ✅ 移动优先

## 常见问题

### Q: 如何在 iPad 上启用文件系统访问？
A: iPad Safari 不支持 File System Access API。使用文件上传功能代替。

### Q: 需要后端服务器吗？
A: 不需要。这是纯前端应用，可以静态部署。

### Q: 如何添加身份验证？
A: 可以在服务器级别添加（HTTP 基础认证或 OAuth）。

### Q: 支持离线吗？
A: 是的，Service Worker 支持离线缓存。

### Q: 数据会同步吗？
A: 不会。所有数据存储在本地浏览器中。

## 故障排除

### Service Worker 未注册
- 检查浏览器控制台
- 确保使用 HTTPS
- 清除浏览器缓存

### Manifest 未识别
- 检查 mime-type 是否为 `application/manifest+json`
- 验证 JSON 格式

### PDF 显示不完整
- 检查 PDF.js CDN
- 尝试下载 PDF.js 本地部署

## 监控和日志

### 服务器日志监控
```bash
# Nginx
tail -f /var/log/nginx/access.log

# Apache
tail -f /var/log/apache2/access.log
```

### 客户端错误监控
在浏览器控制台查看错误信息

## 备份和恢复

### 备份配置
```bash
# 备份所有文件
tar czf backup.tar.gz /var/www/pdf-reader/

# 恢复
tar xzf backup.tar.gz -C /var/www/
```

## 版本更新

1. 备份当前版本
2. 更新文件
3. 测试新功能
4. 清除 Service Worker 缓存（可选）
5. 监控用户反馈

## 技术支持

遇到问题时检查:
1. 浏览器控制台错误
2. 服务器日志
3. Network 标签查看请求
4. 尝试隐私模式测试

## 相关资源

- [MDN PWA 文档](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)
- [PDF.js 文档](https://mozilla.github.io/pdf.js/)
- [Service Worker 文档](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Web App Manifest](https://www.w3.org/TR/appmanifest/)

---

**更新时间**: 2026-01-22
