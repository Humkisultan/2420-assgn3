# Assignment 3 - Part 1

# Server Setup and Configuration Guide

This guide outlines the steps to set up a server with a system user, automated service, and a web server to display system information. 



## Task 1: Create the System User

### Steps:
1. Create a system user `webgen` with a home directory at `/var/lib/webgen` and a non-login shell:
   ```bash
   sudo useradd -r -d /var/lib/webgen -s /usr/bin/nologin webgen
   ```
2. Ensure the `webgen` user owns its home directory and all subdirectories:
   ```bash
   sudo mkdir -p /var/lib/webgen/{bin,HTML}
   sudo chown -R webgen:webgen /var/lib/webgen
   ```

### Directory Structure:
```
/var/lib/webgen/
├── bin/
│   └── generate_index
└── HTML/
    └── index.html
```

### Benefits of Using a System User:
- **Security:** A non-login system user restricts interactive logins, reducing attack surface.
- **Segregation:** Limits access to specific tasks and files, preventing accidental or malicious actions.
- **Best Practices:** Ensures the server adheres to the principle of least privilege.



## Task 2: Automate the Index Generation

### Files:
- `generate-index.service`
- `generate-index.timer`

### Steps:
1. Create the service file at `/etc/systemd/system/generate-index.service`:
   ```ini
   [Unit]
   Description=Generate index.html for system information
   After=network-online.target
   Wants=network-online.target

   [Service]
   User=webgen
   Group=webgen
   ExecStart=/var/lib/webgen/bin/generate_index

   [Install]
   WantedBy=multi-user.target
   ```
2. Create the timer file at `/etc/systemd/system/generate-index.timer`:
   ```ini
   [Unit]
   Description=Run generate-index.service daily at 05:00

   [Timer]
   OnCalendar=*-*-* 05:00:00
   Persistent=true

   [Install]
   WantedBy=timers.target
   ```
3. Enable and start the timer:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable generate-index.timer
   sudo systemctl start generate-index.timer
   ```

### Verification:
- Check the timer status:
  ```bash
  sudo systemctl list-timers --all
  ```
- View service logs:
  ```bash
  sudo journalctl -u generate-index.service
  ```



## Task 3: Configure Nginx

### Steps:
1. Update the main Nginx configuration file to run as the `webgen` user:
   ```bash
   sudo nvim /etc/nginx/nginx.conf
   ```
   Modify the `user` directive:
   ```
   user webgen;
   ```

2. Create a server block configuration file:
   ```bash
   sudo nvim /etc/nginx/conf.d/server-block.conf
   ```
   Contents:
   ```
   server {
       listen 80;
       server_name <your_server_ip>;

       root /var/lib/webgen/HTML;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }

       error_page 404 /404.html;
   }
   ```

3. Test and reload Nginx:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

### Why Use a Separate Server Block?
- **Maintainability:** Keeps the main configuration clean and modular.
- **Best Practices:** Easier to isolate, manage, and troubleshoot individual configurations.

### Checking Nginx:
- View service status:
  ```bash
  sudo systemctl status nginx
  ```
- Test the configuration:
  ```bash
  sudo nginx -t
  ```



## Task 4: Set Up the Firewall

### Steps:
1. Install UFW:
   ```bash
   sudo pacman -S ufw
   ```
2. Configure rules:
   ```bash
   sudo ufw allow ssh
   sudo ufw allow http
   sudo ufw limit ssh
   ```
3. Enable UFW:
   ```bash
   sudo ufw enable
   ```

### Verification:
- Check firewall status:
  ```bash
  sudo ufw status verbose
  ```



## Task 5: Verify the Server

1. Visit your server's IP address in a browser:
   ```
   http://<your_server_ip>
   ```
2. Take a screenshot of the system information page showing your IP and the HTML content.
3. Submit your server's IP address to confirm functionality.



## Notes
- Ensure the `generate_index` script in `/var/lib/webgen/bin` is executable:
  ```bash
  chmod +x /var/lib/webgen/bin/generate_index
  ```
- Use `journalctl` for logs:
  ```bash
  sudo journalctl -u nginx.service
  sudo journalctl -u generate-index.service
  ```
- Test with `curl`:
  ```bash
  curl http://<your_server_ip>
  ```
```
