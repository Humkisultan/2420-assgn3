# Assignment 3 - Part 1

# Server Setup and Configuration

This repository contains the necessary files and instructions to set up and configure a server with the following features:

- Automated generation of an HTML index file via a systemd service and timer.
- Nginx web server configured to serve the generated HTML file.
- UFW firewall rules for security.



## Prerequisites

1. **Server Requirements**:
   - Linux-based server (tested on Arch Linux).
   - Root or sudo access.

2. **Installed Tools**:
   - `git`
   - `nginx`
   - `ufw`
   - Systemd (default in most Linux distributions).



## Setup Instructions

### 1. Clone the Repository
Clone this repository to the server:
```bash
git clone <repository-url> /home/<user>/server-setup
cd /home/<user>/server-setup
```

### 2. Deploy Systemd Service and Timer
#### Files:
- `generate-index.service`
- `generate-index.timer`


#### Steps:
1. Copy the service and timer files to `/etc/systemd/system`:
   ```bash
   sudo cp systemd/generate-index.service /etc/systemd/system/
   sudo cp systemd/generate-index.timer /etc/systemd/system/
   ```
2. Reload systemd to recognize new files:
   ```bash
   sudo systemctl daemon-reload
   ```
3. Enable and start the timer:
   ```bash
   sudo systemctl enable generate-index.timer
   sudo systemctl start generate-index.timer
   ```
4. Test the service directly:
   ```bash
   sudo systemctl start generate-index.service
   ```



### 3. Nginx Configuration
#### Files:
- `nginx.conf` (main configuration file)
- `server-block.conf` (server block configuration)

#### Steps:
1. Replace the default Nginx configuration file:
   ```bash
   sudo cp config/nginx.conf /etc/nginx/nginx.conf
   ```
2. Copy the server block file:
   ```bash
   sudo cp config/server-block.conf /etc/nginx/conf.d/
   ```
3. Test and reload Nginx:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```



### 4. Firewall Configuration
1. Install UFW if not already installed:
   ```bash
   sudo pacman -S ufw
   ```
2. Allow necessary ports:
   ```bash
   sudo ufw allow ssh
   sudo ufw allow http
   ```
3. Enable SSH rate limiting:
   ```bash
   sudo ufw limit ssh
   ```
4. Enable the firewall:
   ```bash
   sudo ufw enable
   ```



### 5. Verify the Setup
1. Confirm that the service generates the `index.html` file:
   ```bash
   ls /var/lib/webgen/HTML
   ```
2. Test Nginx:
   - Visit `http://<server-ip>` in a browser.
   - Use `curl` to verify:
     ```bash
     curl http://<server-ip>
     ```



## Directory Structure

```
server-setup/
├── config/
│   ├── nginx.conf
│   ├── server-block.conf
├── scripts/
│   └── generate_index
├── systemd/
│   ├── generate-index.service
│   ├── generate-index.timer
└── README.md
```



## Notes
- Replace `<server-ip>` with your server's IP address.
- Ensure the `generate_index` script is executable:
  ```bash
  chmod +x scripts/generate_index
  ```
- For troubleshooting, check logs:
  ```bash
  sudo journalctl -u generate-index.service
  sudo journalctl -u nginx.service
  ```
```

