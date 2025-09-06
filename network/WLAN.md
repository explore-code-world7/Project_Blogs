# test skip wall

curl -I https://www.google.com

# means

error: 

```bash
curl: (35) OpenSSL SSL_connect: SSL_ERROR_SYSCALL
```

**2. 更新系统证书**

Ubuntu 升级后证书存储可能过期，导致 SSL 验证失败：

```bash
# 更新证书包
sudo apt update
sudo apt install --reinstall ca-certificates
sudo update-ca-certificates --fresh

# 清理旧缓存
sudo rm -rf /usr/share/ca-certificates/mozilla
```

**3. 检查时间和时区**

SSL 证书验证依赖系统时间，时间偏差会导致握手失败：

```bash
# 查看系统时间是否准确
date

# 同步网络时间（安装NTP服务）
sudo apt install ntpdate
sudo ntpdate pool.ntp.org

# 验证时区配置
timedatectl
```

### **6. 检查 OpenSSL 和 Curl 版本**

升级后可能因版本不兼容导致问题：

```bash
# 查看 OpenSSL 版本
openssl version

# 查看 Curl 版本
curl --version

# 更新到最新版本
sudo apt update && sudo apt upgrade curl openssl
```

## 测试用

### **5. 强制指定 TLS 版本**

某些服务器可能不支持旧版 TLS，强制指定版本尝试：

```bash
# 尝试 TLS 1.2
curl --tlsv1.2 -vI https://example.com

# 尝试 TLS 1.3
curl --tlsv1.3 -vI https://example.com

# 跳过证书验证（仅测试用，不安全！）
curl -k https://example.com
```

---
