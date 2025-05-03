# 🐍 Cowrie SSH/Telnet Honeypot Deployment on Ubuntu

This guide walks you through setting up a secure environment, installing common security tools, deploying Cowrie as an SSH/Telnet honeypot, and monitoring activity. Designed for educational and research purposes.

---

## 🔐 Phase 2: Secure the Environment

### ✅ Update the System
```bash
sudo apt update && sudo apt upgrade -y


### ✅ Create a Non-Root User (Recommended)

```bash
sudo adduser honeypot
sudo usermod -aG sudo honeypot
sudo su - honeypot
```

### ✅ Install Basic Tools

```bash
sudo apt install curl wget git unzip net-tools -y
```

---

## 🧰 Phase 3: Manually Install Kali Tools

> ⚠️ Avoid enabling Kali repos on Ubuntu due to stability risks. Install tools manually using `apt` or GitHub.

### ✅ Install Common Tools

#### 🔍 Nmap

```bash
sudo apt install nmap -y
```

#### 🧪 Wireshark (CLI Only)

```bash
sudo apt install tshark -y
```

#### 🔐 Hydra (Brute-force simulation)

```bash
sudo apt install hydra -y
```

#### 🌐 Nikto (Web Vulnerability Scanner)

```bash
sudo apt install nikto -y
```

#### ⚙️ Metasploit

```bash
sudo apt install curl git libpq-dev -y
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
chmod +x msfinstall
sudo ./msfinstall
```

---

## 🎯 Phase 4: Deploy Cowrie Honeypot

### ✅ Step 1: Install Dependencies

```bash
sudo apt install python3-virtualenv libssl-dev libffi-dev build-essential python3-dev libpython3-dev -y
```

### ✅ Step 2: Clone Cowrie

```bash
cd /opt
sudo git clone https://github.com/cowrie/cowrie.git
sudo chown -R honeypot:honeypot /opt/cowrie
cd cowrie
```

### ✅ Step 3: Setup Virtual Environment

```bash
python3 -m venv cowrie-env
source cowrie-env/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

### ✅ Step 4: Configure Cowrie

```bash
cp etc/cowrie.cfg.dist etc/cowrie.cfg
nano etc/cowrie.cfg
```

Recommended changes:

```ini
hostname = ubuntu-honeypot

[ssh]
enabled = true
listen_port = 2222

[telnet]
enabled = true
listen_port = 2223
```

> Use non-standard ports (e.g., 2222 and 2223) to avoid conflict with real services.

### ✅ Step 5: Start Cowrie

```bash
bin/cowrie start
bin/cowrie status
```

Logs will be available at:

```bash
tail -f var/log/cowrie/cowrie.log
```

---

## ✅ Step 8: Redirect Ports (Optional)

Make Cowrie appear to be running on real ports 22/23:

```bash
# SSH redirection
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222

# Telnet redirection
sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
```

> To persist rules, install `iptables-persistent`.

---

## ✅ Step 9: Enable Auto-Start on Boot (Optional)

Create a systemd service:

```bash
sudo nano /etc/systemd/system/cowrie.service
```

Paste:

```ini
[Unit]
Description=Cowrie SSH Honeypot
After=network.target

[Service]
User=honeypot
WorkingDirectory=/opt/cowrie
ExecStart=/opt/cowrie/cowrie-env/bin/python /opt/cowrie/bin/cowrie start
ExecStop=/opt/cowrie/cowrie-env/bin/python /opt/cowrie/bin/cowrie stop
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable cowrie
sudo systemctl start cowrie
```

---

## ✅ Step 10: Monitor and Analyze Logs

Cowrie logs are located in:

```bash
/opt/cowrie/var/log/cowrie/
```

> You can convert logs to JSON and forward them to ELK Stack or Graylog for analysis.

---

## 📌 Disclaimer

This setup is **for educational and research purposes only**. Unauthorized deployment on live networks or usage for malicious intent is strictly prohibited.

---

## 📚 References

* [Cowrie GitHub Repository](https://github.com/cowrie/cowrie)
* [Metasploit](https://github.com/rapid7/metasploit-framework)

---

## ✍️ Author

Your Name - [GitHub Profile](https://github.com/kvabhaya)

```
