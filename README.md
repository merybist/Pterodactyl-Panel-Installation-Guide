# Pterodactyl Panel & Wings Installation Guide (Tested on Ubuntu 24.04 LTS)

This guide will walk you through installing **Pterodactyl Panel** and **Wings** on an Ubuntu 24.04 LTS server.

---

## Step 0. Connect to your server

```bash
ssh root@your-server-ip
```

---

## Step 1. Install Panel

Update packages:

```bash
apt update && apt upgrade -y
```

Run the installer:

```bash
bash <(curl -s https://pterodactyl-installer.se)
```

Select option **0 (Install Panel)**.

### Configuration prompts

- **Database name**: press Enter (default)
- **Database username**: press Enter (default)
- **Database password**: create a strong password (save it securely)
- **Timezone**: enter your timezone (e.g., `Europe/Kiev`)
- **Email for Let's Encrypt and Pterodactyl**: enter a real, valid email
- **Admin account setup:**
  - Email: `example@mail.com`
  - Username: `admin`
  - First name: `admin`
  - Last name: `admin`
  - Password: `yourStrongPassword`
- **FQDN**: enter your domain (e.g., `panel.example.com`) or server IP if no domain

⚠️ Let's Encrypt **does not work with IP addresses**. Use a domain if you want SSL.

- **Configure UFW firewall?**: `y`

Example configuration summary:

```
##############################################################
* Pterodactyl panel v1.11.11 with nginx on ubuntu
* Database name: panel
* Database user: pterodactyl
* Database password: (hidden)
* Timezone: Europe/Kiev
* Email: example@mail.com
* User email: example@mail.com
* Username: admin
* First name: admin
* Last name: admin
* User password: (hidden)
* Hostname/FQDN: 00.11.22.33
* Configure Firewall? true
* Configure Let's Encrypt? false
* Assume SSL? false
##############################################################
```

When asked:

```
Enable sending anonymous telemetry data? (yes/no) [yes]:
```
Answer: `no`

### Possible issue with nginx

If installation ends with:

```
Job for nginx.service failed because the control process exited with error code.
```

Fix by editing the config:

```bash
nano /etc/nginx/sites-enabled/pterodactyl.conf
```

- Change:
  ```nginx
  fastcgi_pass unix:/run/php/phpX.X-fpm.sock;
  ```
  to:
  ```nginx
  fastcgi_pass 127.0.0.1:9000;
  ```

- Comment out:
  ```nginx
  # fastcgi_buffer_size 16k;
  # fastcgi_buffers 4 16k;
  ```

Test and restart nginx:

```bash
nginx -t
systemctl restart nginx
```

✅ Visit your FQDN or IP in the browser — you should see the Pterodactyl login page.

---

## Step 2. Install Wings

Run installer again:

```bash
bash <(curl -s https://pterodactyl-installer.se)
```

Choose option **1 (Install Wings)**.

### Configuration prompts

- Configure UFW? → `y`
- Configure database user? → `y`
- Allow external MySQL access? → `N`
- Database host username: (default: `pterodactyluser`)
- Database host password: enter a strong one
- Configure HTTPS with Let's Encrypt? → `n`
- Proceed with installation? → `y`

⚠️ Reminder: Let's Encrypt requires a domain (not just an IP).

### If database user already exists

You may see:

```
CREATE USER 'pterodactyluser'@'127.0.0.1' IDENTIFIED BY 'YOUR_PASSWORD'
```

Fix:

```bash
mysql -u root -p
SELECT user, host FROM mysql.user WHERE user='pterodactyluser';
DROP USER 'pterodactyluser'@'127.0.0.1';
exit
```

Then rerun the installer.

---

## 🎭 If the node does not come up

### Possible reasons for this
- You didn't raise the issue yourself
- You received the error `“Range overlap / Pool overlaps”`
- The list may be supplemented...

### Raise the node yourself:

Run command:
```bash
wings --debug
```
💚 Noda got up.

---

⚠️ If you see:
```error
failed to configure docker environment error=Error response from daemon:
invalid pool request: Pool overlaps with other one on this address space
```
Go to - `Fix "Pool overlaps"`

---

### Fix "Pool overlaps":

The problem is that your **“Subnets”** overlap.
And what **Wings** is trying to use is already being used by something else.

Run command:
```bash
nano /etc/pterodactyl/config.yml
```

Find:

```yaml
docker:
  network:
    interface: 172.18.0.1
    dns:
    - 1.1.1.1
    - 1.0.0.1
    name: pterodactyl_nw
    ispn: false
    driver: bridge
    network_mode: pterodactyl_nw
    is_internal: false
    enable_icc: true
    network_mtu: 1500
    interfaces:
      v4:
        subnet: 172.18.0.0/16
        gateway: 172.18.0.1
      v6:
        subnet: fdba:17c8:6c94::/64
        gateway: fdba:17c8:6c94::1011
```
You should be aware of your **busy** and **free** subnets.
After that, establish the **free** ones.

Example:
```yaml
    interfaces:
      v4:
        subnet: 172.18.0.0/16
        gateway: 172.18.0.1
```
We know that the network `“172.18.0.0/16”` is busy.
Let's replace it with a free subnet. For example - `"172.19.0.0/24"`

Now it will look like this:
```yaml
    interfaces:
      v4:
        subnet: 172.19.0.0/24
        gateway: 172.19.0.1
```

Now let us verify the **permissions**

Run commands:
```bash
sudo chown root:root /etc/pterodactyl/config.yml
sudo chmod 640 /etc/pterodactyl/config.yml
```
Delete the old conflicting network (if it was created):
```bash
docker network rm pterodactyl_nw
```
Reload the systemd configuration
```bash
sudo  systemctl daemon-reload
```
Restart Wings:
```bash
sudo systemctl restart wings
```
Make sure that the service is alive and does not crash immediately:
```bash
sudo  systemctl status wings --no-pager
```

### 💚 Congratulations! You have fixed the problem.

---

## Useful commands

Run from inside `/var/www/pterodactyl` directory:

```bash
php artisan p:user:list    # List all users
php artisan p:user:delete  # Delete a user
php artisan p:user:make    # Create a new user
```

---

## 🎉 Congratulations!
You have successfully installed **Pterodactyl Panel** and **Wings** on Ubuntu 24.04 LTS.

## ❗ Need Help?
If you encounter any problems or have questions, please open an **issue** on GitHub.
"""

README powered by **merybist**  
Installer script borrowed from [github.com/pterodactyl-installer/pterodactyl-installer](https://github.com/pterodactyl-installer/pterodactyl-installer)
