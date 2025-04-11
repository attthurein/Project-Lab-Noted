# **Odoo Setup with Cloudflare Tunnel and SSL on Windows 10**

## **Prerequisites**
- **Domain**: Purchased from Your Domain Provider (e.g., `yourdomain.com`) and managed via **Cloudflare**
- **Cloudflare Account** (Free plan)
- **Small Server (24/7)**:
  - Ubuntu VM, Raspberry Pi, or Linux VM (inside the same NAT network as Odoo Services Running Computer)
- **Windows 10 Computer** with a static IP (e.g., `192.168.10.11`)
- **Target VM/Service**: ESXi UI, Ubuntu, etc.
- **Odoo for Windows** (Download from [official site](https://www.odoo.com/page/download))

---

## **1. Download and Install Odoo**
1. Go to [https://www.odoo.com/page/download](https://www.odoo.com/page/download)
2. Download the Windows installer and complete the installation.
3. By default, Odoo runs on `http://localhost:8069`

---

## **2. Set Up Cloudflare Tunnel on a Linux System**

> A Linux system (e.g., Ubuntu VM) on the same local network is required for tunneling.

### **Install Cloudflare Tunnel**
```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

### **Login and Create Tunnel**
```bash
cloudflared tunnel login
cloudflared tunnel create lab-tunnl
```

### **Create Configuration File**
```bash
sudo mkdir -p /etc/cloudflared
sudo nano /etc/cloudflared/config.yml
```

#### **Sample config.yml**
```yaml
tunnel: lab-tunnl
credentials-file: /home/lab-user/.cloudflared/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  - hostname: odoo.yourdomain.com
    service: http://192.168.10.11:8069
  - service: http_status:404

originRequest:
  noTLSVerify: true
```

### **Start Tunnel**
```bash
cloudflared tunnel run lab-tunnl
```

> ⚠️ Replace `192.168.10.11` with your Windows computer's local IP.

---

## **3. Add DNS Entry in Cloudflare**
If not automatically created:
1. Go to **Cloudflare Dashboard** → **DNS**.
2. Add a **CNAME** record:
   - **Name**: `odoo`
   - **Target**: `lab-tunnl.cfargotunnel.com`
   - Enable **Proxy (orange cloud)**

---

## **4. Generate SSL Certificate from Cloudflare**

1. Go to **Cloudflare Dashboard** → **SSL/TLS** → **Origin Server**
2. Click **Create Certificate**
   - **Private Key Type**: RSA
   - **Validity**: 15 years
   - **Hostnames**:
     - `yourdomain.com`
     - `*.yourdomain.com`

3. Save files:
   - **origin.crt**
   - **origin.key**

---

## **5. Install Certificate in Windows**

1. Create a folder:
   ```
   C:\Certs\Cloudflare\
   ```
2. Place:
   - `origin.crt` → `C:\Certs\Cloudflare\origin.crt`
   - `origin.key` → `C:\Certs\Cloudflare\origin.key`

---

## **6. Configure Odoo for SSL**

Edit the Odoo configuration file:  
`C:\Program Files\Odoo 18.0.xxxxx\server\odoo.conf`

Add or update the following:

`proxy_mode = True`

```ini
ssl_certificate = C:\Certs\Cloudflare\origin.crt
ssl_private_key = C:\Certs\Cloudflare\origin.key
```

---

## **7. Restart Odoo Service**

From **Windows Services** or using Command Prompt:
```cmd
net stop odoo
net start odoo
```

---

## ✅ **Result**
Odoo will now be securely accessible via:  
**https://odoo.yourdomain.com**