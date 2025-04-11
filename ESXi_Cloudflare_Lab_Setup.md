
# **ESXi Lab Setup with Ubuntu and Cloudflare**

### **Prerequisites**
- **Domain**: Purchased from Z.com (e.g., `yourdomain.com`)
- **Cloudflare Account** (Free plan)
- **Small Server (24/7)**:
  - Ubuntu VM, Raspberry Pi, or Linux VM (inside the same NAT network as ESXi)
- **ESXi Server** with a static IP (e.g., `192.168.10.10`)
- **Target VM/Service**: ESXi UI, Ubuntu, etc.

---

### **1. Install ESXi Server with Static IP**
1. **Install ESXi** on a physical server or a VM.
2. Set a **static IP** for your ESXi (e.g., `192.168.10.10`).

### **2. Install Ubuntu (or any Linux OS)**
1. Install **Ubuntu** or your preferred Linux OS on a VM (within the same NAT network as ESXi).
2. **Update and Upgrade Dependencies**:
   ```bash
   sudo apt-get update
   sudo apt-get upgrade
   ```

---

### **3. Setup Cloudflare**
1. **Add Domain to Cloudflare**:  
   - Log in to **Cloudflare** and add your domain (e.g., `yourdomain.com`).
   
2. **Change Domain Name Servers to Cloudflare**:
   - Go to **Z.com (GMO)** or your domain provider dashboard.
   - Find **DNS / Nameserver settings**.
   - Replace the current nameservers with Cloudflare's:
     - **ns1**: `mario.ns.cloudflare.com`
     - **ns2**: `lucy.ns.cloudflare.com`
   - Propagation may take up to 30 minutes.

---

### **4. Install Cloudflare Tunnel on Ubuntu**

1. **Download and Install Cloudflare Tunnel**:
   ```bash
   sudo wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
   sudo dpkg -i cloudflared-linux-amd64.deb
   ```

2. **Authenticate and Create Tunnel**:
   ```bash
   cloudflared tunnel login   # Log in and authorize the app with your Cloudflare account
   cloudflared tunnel create lab-tunnl   # Create a new tunnel
   ```

3. **Configure the Tunnel**:
   - Create the necessary directory for Cloudflare configuration:
     ```bash
     sudo mkdir -p /etc/cloudflared
     ```
   - Edit the configuration file (`config.yml`):
     ```bash
     sudo nano /etc/cloudflared/config.yml
     ```
   - **Sample Configuration**:
     ```yaml
     tunnel: lab-tunnl
     credentials-file: /home/lab-user/.cloudflared/14d274ea-2271-4904-89ed-95f7ac17a49f.json

     ingress:
       - hostname: esxi.yourdomain.com   # Replace with your subdomain
         service: https://192.168.10.10:443      # Replace with ESXi Server IP
       - service: http_status:404

     originRequest:
       noTLSVerify: true
     ```

4. **Enable Tunnel on Boot (Optional)**:
   ```bash
   sudo cloudflared service install
   ```

---

### **5. Set up Subdomains and Hosting (Optional)**
- **Managing Subdomains**: 
  - If you want to add subdomains like `esxi.yourdomain.com` or host other services, you can do it directly in **Cloudflare DNS** settings:
    1. **Login to Cloudflare** → Go to **DNS Settings** for your domain.
    2. Add A, CNAME, or other records to point subdomains to the appropriate IP address (or internal services via the tunnel).

- **Hosting Services**: 
  - If you want to host a website or additional services, you can either:
    - **Use Cloudflare Pages** or **Cloudflare Workers** (for static sites)
    - **Host with a 3rd Party Provider** and point subdomains to their servers via DNS records in Cloudflare.

---

### **Where to Control Your Domain**
- **Domain Management (Z.com)**:  
  Z.com controls the **registration** and **nameservers** of your domain. If you want to create subdomains or link your domain to external services (like hosting), you manage DNS records in **Cloudflare**.

- **Subdomain and Hosting Control (Cloudflare)**:  
  Cloudflare controls the **DNS** records, including subdomains, and also allows you to configure proxy services, SSL, and security settings.

---

### **Conclusion**
Your setup includes Cloudflare tunneling for secure, remote access to your internal network (e.g., ESXi) with SSL encryption. You can manage **DNS settings** for subdomains in Cloudflare and use it to proxy other services like hosting or internal servers.
