# 🚀 Node.js Deployment on AWS EC2 with Nginx & PM2

A production-style deployment of a Node.js backend on an AWS EC2 instance — configured with Nginx as a reverse proxy and PM2 for process management.

> This documents a real hands-on lab I completed: provisioning a cloud server from scratch, configuring networking, and deploying a running Node.js API accessible via a public IP.

---

## 🏗️ Architecture

```
Internet (Port 80)
        │
        ▼
   [ AWS EC2 ]
   Ubuntu 24.04
        │
        ▼
   [ Nginx ]  ← Reverse Proxy (Port 80 → 3000)
        │
        ▼
  [ Node.js App ]  ← Managed by PM2 (Port 3000)
```

---

## ⚙️ Tech Stack

| Layer | Tool |
|---|---|
| Cloud Provider | AWS EC2 (Asia Pacific — Mumbai) |
| OS | Ubuntu 24.04 LTS |
| Runtime | Node.js v18.19.1 / npm v9.2.0 |
| Web Server | Nginx v1.24.0 (reverse proxy) |
| Process Manager | PM2 |
| Region | ap-south-1 (Mumbai) |

---

## 📋 What I Did — Step by Step

### 1. AWS Setup
- Launched an **EC2 t2.micro instance** with Ubuntu 24.04 LTS in the Mumbai region
- Configured Security Group to allow inbound traffic on **Port 22** (SSH) and **Port 80** (HTTP)
- Generated a `.pem` key pair for SSH access

### 2. SSH Into the Server
```bash
ssh -i ED25519_Kuha_aws.pem ubuntu@<your-ec2-public-ip>
```

### 3. Install Node.js & npm
```bash
sudo apt update
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
node --version   # v18.19.1
npm --version    # 9.2.0
```

### 4. Install Nginx
```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
nginx -v   # nginx/1.24.0
```

At this point, visiting `http://<your-ec2-ip>` in the browser shows the **Welcome to nginx!** page — server is live.

### 5. Create the Node.js App
```bash
mkdir node-test && cd node-test
npm init -y
```

`index.js`:
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({
    success: true,
    service: 'node-backend',
    port: 3000
  });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

```bash
npm install express
```

### 6. Configure Nginx as Reverse Proxy
```bash
sudo nano /etc/nginx/sites-available/default
```

Replace the default `location /` block with:
```nginx
location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
}
```

Test and reload Nginx:
```bash
sudo nginx -t          # Test config for errors
sudo systemctl reload nginx
```

### 7. Run App with PM2
```bash
sudo npm install -g pm2
pm2 start index.js --name app
pm2 status             # Check it's online
pm2 startup            # Auto-start on server reboot
pm2 save
```

### 8. Verify
Visit `http://<your-ec2-ip>` in the browser:
```json
{
  "success": true,
  "service": "node-backend",
  "port": 3000
}
```

Nginx is receiving traffic on port 80 and proxying it to Node.js on port 3000 — working end to end. ✅

---

## 📸 Screenshots

### EC2 SSH Connection
<img width="648" height="644" alt="image" src="https://github.com/user-attachments/assets/dce3240c-5edb-4ee8-9e90-53d64da78a80" />

### Node.js & Nginx Versions Verified
<img width="831" height="160" alt="image" src="https://github.com/user-attachments/assets/23c44ccb-2864-4ad5-a24a-83a2fdfa3ba1" />

### Nginx Default Page (Server Live)
<img width="1365" height="601" alt="image" src="https://github.com/user-attachments/assets/4415b419-6982-4cfd-bd88-573b7f5b6300" />

### PM2 Status — App Running
<img width="1129" height="218" alt="image" src="https://github.com/user-attachments/assets/cdbfeb97-0218-48b7-9f22-ff600b50ab4d" />


### Node.js API Response via Public IP
<img width="515" height="329" alt="image" src="https://github.com/user-attachments/assets/90639e9a-b96d-40ea-bcb1-47c1d16ade55" />

---

## 💡 Key Concepts Learned

**Why not run Node.js on port 80 directly?**
Port 80 requires root privileges to bind. Instead, Nginx runs as the public-facing server on port 80 and forwards traffic internally to Node.js on port 3000 — safer and more scalable.

**Why PM2?**
If the Node.js process crashes, PM2 automatically restarts it. It also persists across server reboots with `pm2 startup`, which a plain `node index.js` won't do.

**Why IAM user instead of root?**
AWS best practice — root account should never be used for day-to-day tasks. IAM users have scoped permissions, reducing the blast radius of any credential leak.

---

## 🔜 What's Next

- [ ] Add a custom domain with Namecheap + DNS A record
- [ ] Configure SSL/HTTPS with Let's Encrypt (Certbot)
- [ ] Set up GitHub Actions CI/CD to auto-deploy on push
- [ ] Containerize the app with Docker

---

## 📎 Related Projects

- [Weather App](https://github.com/harshkumar-2005/Weather-App) — Live frontend project deployed on Vercel
- [A2Z DSA Solutions](https://github.com/harshkumar-2005/A2Z_Striver_Solution) — C++ DSA practice

---

## 👤 Author

**Harsh Thakur**
[LinkedIn](https://linkedin.com/in/harshkumar-thakur) · [GitHub](https://github.com/harshkumar-2005) · [Email](mailto:kumarharsh93982@gmail.com)
