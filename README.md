# 🧩 Pipe Network - PoP Node

> Minimal setup guide for running a Pipe PoP node on your own server.  
> Dockerless setup using systemd, auto-restart and clean file structure.  
> Project: [https://pipe.network](https://pipe.network) | Docs: [https://docs.pipe.network/devnet-2](https://docs.pipe.network/devnet-2)

---

## ⚙️ VPS Requirements

| Resource   | Minimum         |
|------------|------------------|
| RAM        | 4 GB (8 GB recommended) |
| Disk       | 100 GB (cache)   |
| OS         | Ubuntu 20.04/22.04 |
| Network    | Stable internet connection |
| Ports      | Open 8003, 80, 443 (TCP) |
| IP Support | IPv4 or IPv6     |

---

## 🚀 Installation

### 1. Download latest binary (v0.2.8)

```bash
cd ~
curl -L -o pop "https://dl.pipecdn.app/v0.2.8/pop"
chmod +x ./pop
```

### 2. Create service user and folders

```bash
sudo useradd -r -m -s /sbin/nologin pop-svc-user -d /home/pop-svc-user 2>/dev/null || true
sudo mkdir -p /opt/pop /var/lib/pop /var/cache/pop/download_cache
sudo mv -f ~/pop /opt/pop/
sudo chmod +x /opt/pop/pop
```

### 3. Move your `node_info.json` if you already registered

```bash
sudo mv -f ~/node_info.json /var/lib/pop/ 2>/dev/null || true
```

### 4. Set permissions

```bash
sudo chown -R pop-svc-user:pop-svc-user /var/lib/pop /var/cache/pop /opt/pop
```

### 5. Create systemd service

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
  --pubKey=YOUR_SOLANA_ADDRESS \
  --max-disk=500 \
  --cache-dir=/var/cache/pop/download_cache \
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

### 6. Start and enable the node

```bash
sudo systemctl daemon-reload
sudo systemctl enable pop.service
sudo systemctl start pop.service
```

> 💡 The node runs automatically using `systemd`.  
> You don’t need to launch it manually with `./pop`.

---

## 📊 Useful Commands

### Check node status

```bash
sudo systemctl status pop.service
```

### View logs

```bash
sudo journalctl -u pop.service -n 50 --no-pager
```

---

## 🔁 Upgrade the Node (latest v0.2.8)

```bash
cd ~
curl -L -o pop "https://dl.pipecdn.app/v0.2.8/pop"
chmod +x ./pop
sudo mv ./pop /opt/pop/pop
sudo setcap 'cap_net_bind_service=+ep' /opt/pop/pop
cd /var/lib/pop
/opt/pop/pop --refresh
```

---

## 🎁 Referral (Optional)

### Generate a referral code:

```bash
cd /var/lib/pop
/opt/pop/pop --gen-referral-route
```

(⚠️ May return 500 error if the endpoint is down)

---

## 🔐 Backup

```bash
cp /var/lib/pop/node_info.json ~/node_info.backup
```

---

## 📡 Monitoring

Use the project dashboard:  
👉 [https://dashboard.pipenetwork.com/node-lookup](https://dashboard.pipenetwork.com/node-lookup)

Search your `node_id` from `node_info.json`.

---

## ✅ Tips

- Always allow ports 8003, 80, 443:
```bash
sudo ufw allow 8003/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload
```

- Use this alias for convenience (optional):

```bash
echo "alias pop='cd /var/lib/pop && /opt/pop/pop'" >> ~/.bashrc && source ~/.bashrc
```

---

> Maintained by [tokionode](https://github.com/KARINE001) 💙  
> Feedback and contributions welcome!
