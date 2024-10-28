# 1. Project Origin

Recently, a nearby internet café offered a special promotion: recharge 30 yuan, get 100 extra! With a keen eye for deals, I quickly recharged 30 yuan. However, since I’m not really into gaming these days (there hasn’t been anything that excites me lately), I figured I’d use this chance to code at the café. The curved screen there looked much more comfortable than my 16-inch laptop.

However, I soon discovered that setting up a coding environment in the café was much trickier than on my personal computer. It wasn’t that I lacked technical skills; the café computers blocked some `.exe` or `.msi` installer files, probably to prevent users from installing unauthorized software. Additionally, many official websites were inaccessible, and there was no way to set up a VPN at the café (that’s like trying to "sneak a pipe" in public). Even accessing GitHub was impossible.

Under normal circumstances, I’d just accept this annoyance, but then an idea hit me: what if I could set up an online IDE on my server? That way, I could code anywhere, whether in an internet café or a coffee shop. And so, this project was born.

# 2. Project Outcome

After a bit of tinkering, I successfully set up an online IDE—`code-server`—on my server. Now, I can access my online IDE through a browser, using it just like local VS Code. Even at the internet café, I can comfortably code, debug, and run programs without worrying about environment configurations.

# 3. System Features and Structure

The system architecture primarily consists of the following components:

* **Server Environment**: A Linux-based cloud server that provides stable, efficient, and reliable performance.
* **Reverse Proxy**: Nginx serves as the reverse proxy server, handling HTTP/HTTPS requests.
* **Online IDE**: `code-server` deployment offers an online VS Code experience.
* **Domain and SSL**: A custom domain is configured with SSL certificates from Let’s Encrypt’s Certbot for secure data transmission.
* **Auxiliary Tools**: Tools like `htop` for system resource monitoring, `ufw` for firewall configuration, `yum` for package management, and `nano` for editing configuration files are also used.

# 4. Open-source Projects and Technologies Used

* **[Linux System](https://www.linux.org/pages/download/)**: The operating system on the server, providing a stable and efficient environment.
* **[code-server](https://github.com/coder/code-server)**: The core online IDE, enabling VS Code access via the browser.
* **[Node.js](https://nodejs.org/zh-cn)**: The runtime environment required by `code-server`, supporting server-side JavaScript.
* **[Nginx](https://nginx.org/en/)**: A high-performance HTTP and reverse proxy server for managing client requests and load balancing.
* **[Git](https://git-scm.com/)**: A version control tool for convenient code management and collaboration.
* **htop**: A system monitoring tool for real-time resource usage visualization.
* **ufw**: A firewall management tool to enhance server security.
* **yum**: The package manager for CentOS, facilitating software installation and updates.
* **GNU nano**: A lightweight text editor used for editing configuration files.
* **Certbot**: An automated SSL certificate management tool for acquiring and managing SSL certificates.
* **EPEL**: A repository that provides additional packages for CentOS, extending software availability.
* **[JDK (java-11-openjdk-devel)](https://www.oracle.com/jp/java/technologies/javase/jdk11-archive-downloads.html)**: A Java development kit to support Java program compilation and execution.
* **[GCC](https://gcc.gnu.org/)**: The GNU Compiler Collection, supporting C/C++ program compilation.
* *(Considering adding JDBC and a database, but my server's memory can’t handle it…)*

---

# 2. Project Solution and Detailed Process

## 1. Deploying `code-server`

![](https://zaizai123.cn/wp-content/uploads/2024/10/QQ20241027-193407-1024x649.png)

### **Step 1: Update System and Install Necessary Packages**

```
sudo yum update -y
sudo yum install -y epel-release
sudo yum install -y wget git gcc gcc-c++ make
```

### **Step 2: Install Node.js**

```
curl -sL https://rpm.nodesource.com/setup_14.x | sudo bash -
sudo yum install -y nodejs
```

### **Step 3: Install `code-server`**

```
curl -fsSL https://code-server.dev/install.sh | sh
```

### **Step 4: Configure `code-server`**

Edit the configuration file `~/.config/code-server/config.yaml`:

```
bind-addr: 127.0.0.1:8080
auth: password
password: your_secure_password
cert: false
```

![](https://zaizai123.cn/wp-content/uploads/2024/10/QQ20241027-200705-1024x649.png)

### **Step 5: Set `code-server` as a System Service**

```
sudo systemctl enable --now code-server@$USER
```

## 2. Configuring Nginx Reverse Proxy

### **Step 1: Install Nginx**

```
sudo yum install -y nginx
```

### **Step 2: Configure Nginx**

Edit the Nginx configuration file `/etc/nginx/conf.d/code-server.conf`:

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

### **Step 3: Apply for SSL Certificate**

```
sudo yum install -y certbot python3-certbot-nginx
sudo certbot --nginx -d code.yourdomain.com
```

## 3. Configure Firewall

```
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

## 4. Access the Online IDE

In the browser, visit `https://code.yourdomain.com`, enter the password you set earlier, and start coding away!

---

# 3. Issues Encountered and Solutions

## Issue 1: Unable to Access `code-server` Service

**Symptom**: `code-server` started normally, but it was inaccessible in the browser.

**Solution**: Check the bind address of `code-server` to ensure it’s set to `127.0.0.1`, and confirm that Nginx’s reverse proxy configuration is correct. Also, verify that the firewall settings allow access to relevant ports.

## Issue 2: SSL Certificate Request Failure

**Symptom**: Certificate validation failed when attempting to use Certbot to request an SSL certificate.

**Solution**: Ensure domain name resolution is correct and that DNS records have propagated. Check the Nginx configuration to verify that the `server_name` is set correctly and that the `/.well-known/acme-challenge/` path is accessible.

## Issue 3: Unable to Access GitHub in the Café

**Symptom**: GitHub and other essential development websites were inaccessible in the café environment.

**Solution**: By setting up an online IDE on my server and using SSH authentication, I bypassed the limitations of the local network. All operations are completed on the server, requiring only a browser.

---

# 4. Project Summary

This project’s success means I can easily code and manage projects from anywhere, free from the constraints of a local development environment. Whether at an internet café, coffee shop, or friend’s place, I can now jump into coding as long as there’s an internet connection.

## Pros

* **Flexibility**: Access the development environment anytime, anywhere, removing time and space constraints.
* **Consistency**: A unified development environment eliminates the hassle of configuring setups on different machines.
* **Security**: HTTPS and password protection ensure the security of my code.

## Cons

* **Performance Depends on Network**: A stable network connection is necessary, as poor connectivity affects the experience.
* **Resource Constraints**: Limited by server memory and performance, making it challenging to handle heavy workloads.

**Friendly Tip**: Due to limited server resources, this is not open to the public yet. If you're interested, feel free to contact me for the URL and password—I’d be happy to share this convenience and joy with you!

---

# Wishing everyone a joyful day, every day!
