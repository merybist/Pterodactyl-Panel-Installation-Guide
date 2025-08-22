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
Installer script borrowed from [github.com/eDenxGT](https://github.com/eDenxGT)
