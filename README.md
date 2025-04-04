# 🖥️**Pipe Network PoP Node**🖥️

This step-by-step guide will walk you through the process of installing, configuring, and running a **Pipe Network PoP Node** on your VPS. It covers both a **quick installation** method and how to set up the node using **systemd** for better management.

---

📍### **VPS Requirements**📍

Before starting, ensure your VPS meets the following requirements for optimal performance:

- **RAM**: Minimum of 4GB (configurable), more RAM is better for higher rewards.
- **Disk Storage**: At least 100GB free disk space (configurable), but 200-500GB is the sweet spot for optimal performance.
- **Internet**: Ensure your VPS has 24/7 internet connectivity for uninterrupted operation.
- **CPU**: 2 vCPUs (or more recommended for better performance).

---

### 💫✅**Step 1: Prepare the VPS**

1. **Update your system packages**:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Install UFW to manage the firewall**:

   ```bash
   sudo apt install ufw -y
   ```

3. **Install `netcat` (optional but recommended for testing external connections)**:

   ```bash
   sudo apt install netcat -y
   ```

4. **Open the required ports (8003, 443, and 80)**:

   ```bash
   sudo ufw allow 8003/tcp
   sudo ufw allow 443/tcp
   sudo ufw allow 80/tcp
   sudo ufw enable
   ```

---

### 💫✅**Step 2: Download the Pipe PoP Binary (Quick Start)**

1. **Download the compiled `pop` binary** from Pipe Network:

   ```bash
   curl -L -o pop "https://dl.pipecdn.app/v0.2.6/pop"
   ```

2. **Assign executable permission to the `pop` binary**:

   ```bash
   chmod +x pop
   ```

3. **Create a directory for the node's cache**:

   ```bash
   mkdir download_cache
   ```

---

### 💫✅**Step 3: Start the Pipe PoP Node (Quick Start)**

1. **Run the node** (without systemd):

   ```bash
   sudo ./pop \
     --ram 8 \              # RAM in GB (adjust to your VPS capacity)
     --max-disk 500 \       # Max disk usage in GB (adjust as necessary)
     --cache-dir /data \    # Cache location (choose a valid path)
     --pubKey <KEY>         # Solana public key
   ```

   **Note:**
   - Replace `<KEY>` with your **Solana public key**. You can retrieve this from your **Solana wallet** (e.g., Phantom, Backpack).
   - **RAM** is set to **8GB** in this example. Adjust this according to the resources available on your VPS.
   - **Max disk** usage is set to **500GB**; you can change it based on your setup.
   - **Cache location** should be a valid directory on your system, such as `/data`, or the directory you created earlier (`download_cache`).
   - **Solana Public Key** is needed to link your node with your Solana wallet and is required for receiving rewards.

2. **If you prefer not to use systemd**, running the node **with sudo** will allow it to bind to ports **80** and **443**, starting immediately.

---

### 💫✅**Step 4: Node Configuration & Monitoring**

#### **Monitor Metrics**

1. **View the node’s metrics**:

   ```bash
   ./pop --status
   ```

2. **Check points** (Note: Points are not active yet):

   ```bash
   ./pop --points-route
   ```

#### **Referral System** (Optional)

1. **Generate a referral**:

   ```bash
   ./pop --gen-referral-route
   ```

2. **Sign up with a referral code** (if applicable):

   ```bash
   sudo ./pop --signup-by-referral-route <CODE>
   ```

---

### 💫✅**Step 5: Run Pipe PoP Node with Systemd (Recommended)**

For better management, we recommend running the node using **systemd**, which ensures it runs in the background and automatically restarts after a reboot.

#### **Create a Dedicated User for the Node**

1. **Create the service user** (`pop-svc-user`):

   ```bash
   sudo useradd -r -m -s /sbin/nologin pop-svc-user -d /home/pop-svc-user 2>/dev/null || true
   ```

#### **Create Required Directories**

1. **Create necessary directories for the node**:

   ```bash
   sudo mkdir -p /opt/pop
   sudo mkdir -p /var/lib/pop
   sudo mkdir -p /var/cache/pop/download_cache
   ```

#### **Move and Set Permissions for Files**

1. **Move the `pop` binary to the installation directory**:

   ```bash
   sudo mv -f ~/pop /opt/pop/
   sudo chmod +x /opt/pop/pop
   ```

2. **Move the `node_info.json` file** to `/var/lib/pop/` if available:

   ```bash
   sudo mv -f ~/node_info.json /var/lib/pop/ 2>/dev/null || true
   ```

3. **Set ownership** for the files:

   ```bash
   sudo chown -R pop-svc-user:pop-svc-user /var/lib/pop
   sudo chown -R pop-svc-user:pop-svc-user /var/cache/pop
   sudo chown -R pop-svc-user:pop-svc-user /opt/pop
   ```

#### **Create the Systemd Service File**

1. **Create the `pop.service` systemd unit file**:

   ```bash
   sudo tee /etc/systemd/system/pop.service << 'EOF'
   [Unit]
   Description=Pipe POP Node Service
   After=network.target
   Wants=network-online.target

   [Service]
   AmbientCapabilities=CAP_NET_BIND_SERVICE
   CapabilityBoundingSet=CAP_NET_BIND_SERVICE
   User=pop-svc-user
   Group=pop-svc-user
   ExecStart=/opt/pop/pop \
       --ram=8 \
       --pubKey <YOUR_SOLANA_PUBKEY> \
       --max-disk 500 \
       --cache-dir /var/cache/pop/download_cache \
       --no-prompt
   Restart=always
   RestartSec=5
   LimitNOFILE=65536
   LimitNPROC=4096
   StandardOutput=journal
   StandardError=journal
   SyslogIdentifier=pop-node
   WorkingDirectory=/var/lib/pop

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

   Replace `<YOUR_SOLANA_PUBKEY>` with your **Solana public key**.

2. **Create a symlink for easy access**:

   ```bash
   ln -sf /var/lib/pop/node_info.json ~/node_info.json
   ```

3. **Set up an alias for convenience**:

   ```bash
   grep -q "alias pop='cd /var/lib/pop && /opt/pop/pop'" ~/.bashrc || echo "alias pop='cd /var/lib/pop && /opt/pop/pop'" >> ~/.bashrc && source ~/.bashrc
   ```

#### **Enable and Start the Service**

1. **Reload systemd, enable the service, and start the node**:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable pop.service
   sudo systemctl start pop.service
   ```

2. **Check the status of the node**:

   ```bash
   sudo systemctl status pop.service
   ```

   The service should show as **active (running)**.

---

### 💫✅**Step 6: Monitoring the Node**

1. **Monitor the node’s logs**:

   ```bash
   sudo journalctl -u pop.service -f
   ```

2. **Check if the node is listening on port 8003**:

   ```bash
   sudo netstat -tulnp | grep 8003
   ```

   You should see:

   ```bash
   tcp        0      0 0.0.0.0:8003            0.0.0.0:*               LISTEN      <pop_pid>/pop
   ```

3. **Verify external connection** (check if port 8003 is accessible):

   ```bash
   nc -zv your-vps-ip 8003
   ```

   You should see:

   ```bash
   Connection to your-vps-ip 8003 port [tcp/*] succeeded!
   ```

---

### 💫✅**Step 7: Troubleshooting Common Issues**

#### **1. Check the Firewall (UFW) Configuration**

Ensure that the necessary ports are open on your VPS:

```bash
sudo ufw status
```

Expected output should show the allowed ports:

```bash
Status: active
8003/tcp                  ALLOW       Anywhere
443/tcp                   ALLOW       Anywhere
80/tcp                    ALLOW       Anywhere
```

If ports are missing, re-add them:

```bash
sudo ufw allow 8003/tcp
sudo

 ufw allow 443/tcp
sudo ufw allow 80/tcp
sudo ufw reload
```

Then restart the service:

```bash
sudo systemctl restart pop.service
sudo systemctl status pop.service
```

#### **2. Check if the Node is Accessible Externally**

If you're having trouble connecting to your VPS from outside, make sure that no additional security group or network configuration (from your VPS provider) is blocking these ports.

For **most VPS**, ports 8003, 443, and 80 should be directly reachable once the firewall is configured correctly. Test this by checking if the port is open:

```bash
nc -zv your-vps-ip 8003
```

Replace `your-vps-ip` with your actual VPS IP address. If the port is accessible, you should see:

```bash
Connection to your-vps-ip 8003 port [tcp/*] succeeded!
```

If this command fails, double-check your **VPS provider's firewall** settings (such as security groups) and make sure the ports are allowed.

---

### 💫✅**Step 8: Use Pipe Network Dashboard for Monitoring**

You can track your node’s performance on the **Pipe Network Dashboard**:

- Visit [Pipe Network Node Lookup](https://dashboard.pipenetwork.com/node-lookup).
- Enter your **Node ID** (found in logs after starting the node) to view your node's current status.

---

### 💥💥**Final Thoughts**💥💥

- **Node ID**: This unique ID identifies your node. Keep it safe for future use.
- **Solana Wallet Address**: Register your node with a Solana wallet to ensure you can receive rewards.
- **Systemd Service**: Using systemd ensures your node runs continuously, even after server reboots.

✅✅✅By following this guide, your Pipe PoP node will be properly set up and monitored, with automatic startup and easy management! 🚀

---


