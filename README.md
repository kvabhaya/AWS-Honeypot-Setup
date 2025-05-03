# 🛡️ Cowrie Honeypot on AWS Free Tier

This project sets up a lightweight SSH/Telnet honeypot using [Cowrie](https://github.com/cowrie/cowrie) on a free-tier AWS EC2 Ubuntu instance. It includes environment hardening, tool installation, port redirection, and optional log monitoring.

---

## 📌 Phase 1: Launch an EC2 Instance

1. **Login to AWS Console**  
   [https://console.aws.amazon.com](https://console.aws.amazon.com)

2. **Create EC2 Instance**
   - Name: `Honeypot-Instance`
   - AMI: **Ubuntu Server 22.04 LTS (Free Tier)**
   - Instance Type: `t2.micro`
   - Key Pair: Select or create new
   - Security Group:
     - `SSH (22)` – Your IP only
     - `HTTP (80)` – Anywhere
     - Add other ports as needed (e.g., `21`, `23`)
   - Storage: 8GB (default)

3. **Launch & Connect via SSH**

   ```bash
   ssh -i your-key.pem ubuntu@<EC2-PUBLIC-IP>
   ```

---

## 🔐 Phase 2: Secure the Environment

1. **Update System**

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **(Optional) Create Non-root User**

   ```bash
   sudo adduser honeypot
   sudo usermod -aG sudo honeypot
   sudo su - honeypot
   ```

3. **Install Basic Tools**

   ```bash
   sudo apt install curl wget git unzip net-tools -y
   ```

---

## 🧰 Phase 3: Install Kali Tools (Selective)

> Avoid adding Kali repos to Ubuntu for stability.

### Recommended Tools:

* **Nmap**

  ```bash
  sudo apt install nmap -y
  ```

* **TShark (Wireshark CLI)**

  ```bash
  sudo apt install tshark -y
  ```

* **Hydra**

  ```bash
  sudo apt install hydra -y
  ```

* **Nikto**

  ```bash
  sudo apt install nikto -y
  ```

* **Metasploit Framework**

  ```bash
  sudo apt install curl git libpq-dev -y
  curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
  chmod +x msfinstall
  sudo ./msfinstall
  ```

---

## 🎯 Phase 4: Deploy Cowrie Honeypot

1. **Install Dependencies**

   ```bash
   sudo apt install python3-virtualenv libssl-dev libffi-dev build-essential python3-dev libpython3-dev -y
   ```

2. **Clone Cowrie**

   ```bash
   cd /opt
   sudo git clone https://github.com/cowrie/cowrie.git
   sudo chown -R honeypot:honeypot cowrie
   cd cowrie
   ```

3. **Setup Python Virtual Environment**

   ```bash
   python3 -m venv cowrie-env
   source cowrie-env/bin/activate
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

4. **Configure Cowrie**

   ```bash
   cp etc/cowrie.cfg.dist etc/cowrie.cfg
   nano etc/cowrie.cfg
   ```

   Recommended Changes:

   ```ini
   hostname = ubuntu-honeypot

   [ssh]
   enabled = true
   listen_port = 2222

   [telnet]
   enabled = true
   listen_port = 2223
   ```

5. **Start Cowrie**

   ```bash
   bin/cowrie start
   bin/cowrie status
   ```

---

## 🔍 Phase 5: Monitor Activity

```bash
tail -f /opt/cowrie/var/log/cowrie/cowrie.log
```

---

## ✅ Optional: Redirect Real Ports to Honeypot

```bash
# Redirect real SSH (22) to Cowrie's 2222
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222

# Redirect Telnet (23) to Cowrie's 2223
sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223
```

Make persistent with:

```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

---

## 🚀 Auto-Start Cowrie on Boot

1. **Create systemd Service**

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

2. **Enable the Service**

   ```bash
   sudo systemctl daemon-reexec
   sudo systemctl daemon-reload
   sudo systemctl enable cowrie
   sudo systemctl start cowrie
   ```

---

## 📦 Logs Location

```bash
/opt/cowrie/var/log/cowrie/
```

You can forward logs to ELK, S3, or other platforms for better analysis.

---

## 📣 Want Alerting or Dashboards?

You can set up:

* Email/Telegram alerts
* Log forwarding to ELK/Graylog
* AWS CloudWatch integration

---

## 🙋‍♂️ Contributing

Feel free to fork and contribute via pull requests. Suggestions for new honeypots, log processing tools, or dashboards are welcome!

```
