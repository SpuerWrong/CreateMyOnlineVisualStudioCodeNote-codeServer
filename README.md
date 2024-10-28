![](https://zaizai123.cn/wp-content/uploads/2024/10/image-10-1024x550.png)

## 一、项目的诞生缘由

最近，家附近的网吧搞活动：充30送100！本着薅羊毛的原则，我果断充值了30元。但我又没有玩游戏的爱好（毕竟最近没有什么能让我热血沸腾的游戏了）。就想着在网吧打会代码吧，毕竟网吧的曲面屏看起来比我那16寸的小笔记本舒服多了。

然而，我很快发现，在网吧搭建编程环境比在自己电脑上搭建要困难得多。不是我技术不行哈，而是网吧的电脑会屏蔽一些.exe或者.msi类型的安装文件，估计是为了防止用户乱装软件。另外，很多软件官网在网吧都无法访问，你总不能在网吧电脑上“科学上网”吧（那和在公共场所”导管“也没区别）？这就导致连GitHub都打不开。

这搁在平时我怒一下也就怒一下了，可我就突然有了一个好点子：我能否在自己的服务器上搭建一个线上IDE呢？这样我就可以在任何地方，无论是网吧还是咖啡店，都能方便地撸代码了！于是，这个项目就这样诞生了。

## 二、项目实现的结果

经过一番折腾，我成功地在自己的服务器上搭建了一个线上IDE——`code-server`。现在，我可以通过浏览器访问我的线上IDE，就像使用本地的VS Code一样。即使在网吧，我也能畅快地编写、调试和运行代码，再也不用担心环境配置的问题了。

## 三、系统功能和相关结构

整个系统架构主要由以下部分组成：

* **服务器环境**：基于Linux系统的云服务器，运行稳定，高效可靠。
* **反向代理**：使用Nginx作为反向代理服务器，处理HTTP/HTTPS请求。
* **线上IDE**：部署`code-server`，提供VS Code的线上使用体验。
* **域名与SSL**：配置了自己的域名，并使用Let's Encrypt的Certbot申请SSL证书，确保数据传输的安全性。
* **辅助工具**：使用了`htop`监控系统资源，`ufw`配置防火墙，`yum`进行软件包管理，`nano`编辑器修改配置文件等。

## 四、采用的开源项目和技术

* **[Linux系统](https://www.linux.org/pages/download/)**：服务器的操作系统，提供了高效稳定的运行环境。
* **[code-server](https://github.com/coder/code-server)**：核心的线上IDE，实现了在浏览器中使用VS Code的功能。
* **[Node.js](https://nodejs.org/zh-cn)**：`code-server`依赖的运行环境，支持JavaScript的服务器端运行。
* **[Nginx](https://nginx.org/en/)**：高性能的HTTP和反向代理服务器，处理客户端的请求和负载均衡。
* **[Git](https://git-scm.com/)**：版本控制工具，方便代码的管理和协作。
* **htop**：系统监控工具，实时查看服务器的资源使用情况。
* **ufw**：防火墙管理工具，增强服务器的安全性。
* **yum**：CentOS的包管理器，方便安装和更新软件包。
* **GNU nano**：轻量级文本编辑器，用于编辑配置文件。
* **Certbot**：自动化的SSL证书获取工具，申请和管理SSL证书。
* **EPEL**：为CentOS提供额外软件包的仓库，扩展了软件的可用性。
* **[JDK (java-11-openjdk-devel)](https://www.oracle.com/jp/java/technologies/javase/jdk11-archive-downloads.html)**：Java开发工具包，支持Java程序的编译和运行。
* **[GCC](https://gcc.gnu.org/)**：GNU编译器套件，支持C/C++程序的编译。
* (我还打算加JDBC和数据库，但我的服务器内存不允许啊......)

---

# 二、项目的解决方案与具体流程

## 1. 部署`code-server`

![](https://zaizai123.cn/wp-content/uploads/2024/10/QQ20241027-193407-1024x649.png)

### **步骤1：更新系统并安装必要的软件包**

```
sudo yum update -y
sudo yum install -y epel-release
sudo yum install -y wget git gcc gcc-c++ make
```

### **步骤2：安装Node.js**

```
curl -sL https://rpm.nodesource.com/setup_14.x | sudo bash -
sudo yum install -y nodejs
```

### **步骤3：安装`code-server`**

```
curl -fsSL https://code-server.dev/install.sh | sh
```

### **步骤4：配置`code-server`**

修改配置文件`~/.config/code-server/config.yaml`：

```
bind-addr: 127.0.0.1:8080
auth: password
password: 你的安全密码
cert: false
```

![](https://zaizai123.cn/wp-content/uploads/2024/10/QQ20241027-200705-1024x649.png)

### **步骤5：设置`code-server`为系统服务**

```
sudo systemctl enable --now code-server@$USER
```

## 2. 配置Nginx反向代理

### **步骤1：安装Nginx**

```
sudo yum install -y nginx
```

### **步骤2：配置Nginx**

编辑Nginx配置文件`/etc/nginx/conf.d/code-server.conf`：

```
server {
    listen 80;
    server_name code.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Accept-Encoding gzip;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

![](https://zaizai123.cn/wp-content/uploads/2024/10/QQ20241027-201637-1024x649.png)

### **步骤3：申请SSL证书**

```
sudo yum install -y certbot python3-certbot-nginx
sudo certbot --nginx -d code.yourdomain.com
```

## 3. 配置防火墙

```
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

## 4. 访问线上IDE

在浏览器中访问`https://code.yourdomain.com`，输入之前设置的密码，即可开始愉快地编码之旅！giao

---

# 三、开发过程中遇到的问题和解决方案

## 问题1：无法访问`code-server`服务

**症状**：`code-server`启动正常，但在浏览器中无法访问。

**解决方案**：检查`code-server`的绑定地址，确保它绑定在`127.0.0.1`上，并且Nginx的反向代理配置正确。同时，确认防火墙设置允许相关端口的访问。

## 问题2：SSL证书申请失败

**症状**：使用Certbot申请SSL证书时，出现验证失败的错误。

**解决方案**：确保域名解析正确，DNS记录已经生效。检查Nginx配置，确保`server_name`配置正确，并且`/.well-known/acme-challenge/`路径可以被正常访问。

## 问题3：在网吧无法访问GitHub

**症状**：在网吧环境下，无法访问GitHub等开发必备网站。

**解决方案**：通过在自己的服务器上搭建线上IDE，使用SSH认证，避开了本地网络环境的限制，所有操作都在服务器上完成，只需要一个浏览器即可。

---

# 四、项目总结

这个项目的成功，让我在任何地方都能方便地编写和管理代码，不再受限于本地的开发环境。无论是在网吧、咖啡店，还是在朋友家，只要有网络，我都能随时开启我的编码之旅。

## 优势

* **灵活性**：随时随地访问开发环境，打破了时间和空间的限制。
* **一致性**：统一的开发环境，避免了在不同机器上配置环境的麻烦。
* **安全性**：通过HTTPS和密码保护，确保了代码的安全。

## 劣势

* **性能依赖网络**：需要稳定的网络连接，网络不佳时体验会受到影响。
* **资源受限**：受服务器内存和性能限制，无法承担过于繁重的任务。

**友情提示**：由于服务器资源有限，暂不对外开放。如果你感兴趣，欢迎联系我索取网址和密码，我非常乐意与你分享这份便利和快乐！

---

# 最后希望大家每天开心
