# MySQL Installation Guide

This guide provides comprehensive step-by-step instructions for installing MySQL on Windows and Linux operating systems.

---

## Table of Contents
1. [MySQL Installation on Windows](#mysql-installation-on-windows)
2. [MySQL Installation on Linux](#mysql-installation-on-linux)
3. [Post-Installation Configuration](#post-installation-configuration)
4. [Verification](#verification)

---

## MySQL Installation on Windows

### Prerequisites
- Windows 10 or later (64-bit recommended)
- Administrator privileges
- At least 4GB of available disk space
- Internet connection for downloading MySQL installer

### Step-by-Step Installation

#### Step 1: Download MySQL Installer
1. Open your web browser and navigate to the official MySQL website: https://dev.mysql.com/downloads/installer/
2. You'll see two installer options:
   - **mysql-installer-web-community** (smaller, downloads components during installation)
   - **mysql-installer-community** (larger, includes all components)
3. Click **Download** on your preferred installer (web installer recommended for most users)
4. You may be prompted to sign up or log in - click **"No thanks, just start my download"** at the bottom if you prefer to skip registration
5. Save the installer file (typically named `mysql-installer-community-x.x.x.msi`) to your Downloads folder

#### Step 2: Run the MySQL Installer
1. Navigate to your Downloads folder
2. Double-click the `mysql-installer-community-x.x.x.msi` file
3. If prompted by User Account Control (UAC), click **Yes** to allow the installer to make changes

#### Step 3: Choose Setup Type
1. The MySQL Installer window will open with setup type options:
   - **Developer Default**: Installs MySQL Server, MySQL Shell, MySQL Workbench, connectors, and documentation
   - **Server Only**: Installs only the MySQL Server
   - **Client Only**: Installs client programs without the server
   - **Full**: Installs all available MySQL products
   - **Custom**: Allows you to select specific components

2. For learning SQL, select **Developer Default**
3. Click **Next**

#### Step 4: Check Requirements
1. The installer will check for required software (Visual C++ Redistributables, etc.)
2. If any requirements are missing, click **Execute** to install them automatically
3. Once all requirements are met, click **Next**

#### Step 5: Installation
1. Review the list of products to be installed
2. Click **Execute** to begin the installation
3. Wait for all components to install (this may take several minutes)
4. Once completed, all items will show a green checkmark
5. Click **Next**

#### Step 6: Product Configuration - MySQL Server
1. The configuration wizard will start automatically
2. **Type and Networking** configuration:
   - Config Type: Select **Development Computer** (uses minimal memory)
   - Connectivity: Leave default settings:
     - TCP/IP enabled
     - Port: **3306** (default MySQL port)
     - Open Windows Firewall port: **Checked**
   - Click **Next**

3. **Authentication Method**:
   - Select **Use Strong Password Encryption for Authentication (RECOMMENDED)**
   - This uses the newer caching_sha2_password authentication
   - Click **Next**

4. **Accounts and Roles**:
   - Enter a **Root Password** (this is the administrator password - REMEMBER THIS!)
   - Confirm the password
   - (Optional) Click **Add User** to create additional user accounts
   - For learning purposes, you can proceed with just the root account
   - Click **Next**

5. **Windows Service**:
   - Configure MySQL Server as Windows Service: **Checked**
   - Windows Service Name: **MySQL80** (or current version number)
   - Start the MySQL Server at System Startup: **Checked** (recommended)
   - Run Windows Service as: Select **Standard System Account**
   - Click **Next**

6. **Server File Permissions**:
   - Select **Yes, grant full access to the user running the Windows Service**
   - Click **Next**

7. **Apply Configuration**:
   - Click **Execute** to apply all configuration settings
   - Wait for all steps to complete (green checkmarks)
   - Click **Finish**

#### Step 7: Product Configuration - Additional Products
1. If you installed MySQL Router, Samples and Examples, or other products, follow the configuration wizards for each
2. For most users, you can accept the default settings
3. Click **Next** through each configuration

#### Step 8: Complete Installation
1. Once all products are configured, you'll see the "Installation Complete" screen
2. You can check **Start MySQL Workbench after Setup** if you want to start using the GUI immediately
3. Click **Finish**

#### Step 9: Add MySQL to System PATH (Optional but Recommended)
1. Right-click on **This PC** or **My Computer** and select **Properties**
2. Click on **Advanced system settings**
3. Click **Environment Variables**
4. Under **System variables**, find and select **Path**, then click **Edit**
5. Click **New** and add the MySQL bin directory path:
   - Default path: `C:\Program Files\MySQL\MySQL Server 8.0\bin`
6. Click **OK** on all windows to save
7. Restart any open Command Prompt or PowerShell windows

---

## MySQL Installation on Linux

MySQL can be installed on various Linux distributions. Below are instructions for the most common distributions.

### Option A: Ubuntu/Debian-based Systems

#### Prerequisites
- Ubuntu 20.04 or later / Debian 10 or later
- Root or sudo privileges
- Internet connection

#### Step 1: Update Package Index
```bash
sudo apt update
```

#### Step 2: Install MySQL Server
```bash
sudo apt install mysql-server
```
- Enter **Y** when prompted to confirm installation
- The installation will download and install MySQL Server and dependencies

#### Step 3: Check MySQL Service Status
```bash
sudo systemctl status mysql
```
- You should see "active (running)" in green
- Press **q** to exit the status view

#### Step 4: Start MySQL Service (if not running)
```bash
sudo systemctl start mysql
```

#### Step 5: Enable MySQL to Start on Boot
```bash
sudo systemctl enable mysql
```

#### Step 6: Secure MySQL Installation
```bash
sudo mysql_secure_installation
```

Follow the prompts:
1. **VALIDATE PASSWORD COMPONENT**: 
   - Type **Y** to enable password validation (recommended for production)
   - Type **N** for learning/development environments
   
2. **Password Validation Policy** (if enabled):
   - Select level: 0 (LOW), 1 (MEDIUM), or 2 (STRONG)
   
3. **Set root password**:
   - Enter a strong password for the root user
   - Confirm the password
   
4. **Remove anonymous users?**
   - Type **Y** (recommended)
   
5. **Disallow root login remotely?**
   - Type **Y** for production (more secure)
   - Type **N** for development if you need remote access
   
6. **Remove test database?**
   - Type **Y** (recommended)
   
7. **Reload privilege tables now?**
   - Type **Y**

#### Step 7: Adjust Authentication Method (Ubuntu 22.04+)
On newer Ubuntu versions, you may need to adjust the root user authentication:

```bash
sudo mysql
```

Then run the following SQL commands:
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_password';
FLUSH PRIVILEGES;
EXIT;
```

---

### Option B: RHEL/CentOS/Fedora-based Systems

#### Prerequisites
- RHEL 8+ / CentOS 8+ / Fedora 34+
- Root or sudo privileges
- Internet connection

#### Step 1: Download MySQL Repository
For RHEL/CentOS 8:
```bash
sudo dnf install https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm
```

For Fedora:
```bash
sudo dnf install https://dev.mysql.com/get/mysql80-community-release-fc34-1.noarch.rpm
```

#### Step 2: Install MySQL Server
```bash
sudo dnf install mysql-server
```
- Type **Y** when prompted to confirm installation

#### Step 3: Start MySQL Service
```bash
sudo systemctl start mysqld
```

#### Step 4: Enable MySQL to Start on Boot
```bash
sudo systemctl enable mysqld
```

#### Step 5: Check MySQL Service Status
```bash
sudo systemctl status mysqld
```

#### Step 6: Retrieve Temporary Root Password
MySQL generates a temporary root password during installation:
```bash
sudo grep 'temporary password' /var/log/mysqld.log
```
- Copy this password; you'll need it in the next step

#### Step 7: Secure MySQL Installation
```bash
sudo mysql_secure_installation
```

Follow the prompts:
1. Enter the **temporary root password** you retrieved
2. Set a **new root password** (must meet complexity requirements)
3. Answer **Y** to all subsequent questions (remove anonymous users, disallow remote root login, remove test database, reload privileges)

---

### Option C: Arch Linux

#### Step 1: Update System
```bash
sudo pacman -Syu
```

#### Step 2: Install MySQL (MariaDB)
Arch Linux uses MariaDB (MySQL fork) in official repositories:
```bash
sudo pacman -S mariadb
```

#### Step 3: Initialize MySQL Data Directory
```bash
sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql
```

#### Step 4: Start MySQL Service
```bash
sudo systemctl start mariadb
```

#### Step 5: Enable MySQL to Start on Boot
```bash
sudo systemctl enable mariadb
```

#### Step 6: Secure Installation
```bash
sudo mysql_secure_installation
```
- Follow the prompts similar to Ubuntu instructions above

---

## Post-Installation Configuration

### Create a New User (Recommended for Development)

1. Log in to MySQL as root:
   ```bash
   mysql -u root -p
   ```
   - Enter your root password when prompted

2. Create a new user:
   ```sql
   CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'strong_password';
   ```

3. Grant privileges:
   ```sql
   GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'localhost' WITH GRANT OPTION;
   FLUSH PRIVILEGES;
   ```

4. Exit MySQL:
   ```sql
   EXIT;
   ```

### Configure MySQL for Remote Access (Optional)

**Linux:**
1. Edit the MySQL configuration file:
   ```bash
   sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
   ```
   
2. Find the line `bind-address = 127.0.0.1` and change it to:
   ```
   bind-address = 0.0.0.0
   ```

3. Save and exit (Ctrl+X, then Y, then Enter)

4. Restart MySQL:
   ```bash
   sudo systemctl restart mysql
   ```

**Windows:**
1. Open MySQL Workbench or MySQL Command Line Client
2. Execute: `SET GLOBAL bind_address = '0.0.0.0';`
3. Restart MySQL service from Services panel

---

## Verification

### Verify MySQL Installation

**Windows (Command Prompt or PowerShell):**
```cmd
mysql --version
```

**Linux (Terminal):**
```bash
mysql --version
```

Expected output: `mysql  Ver 8.x.x for [OS] on [architecture]`

### Test MySQL Connection

**Connect to MySQL:**
```bash
mysql -u root -p
```
- Enter your root password

**Once connected, run a test query:**
```sql
SELECT VERSION();
SHOW DATABASES;
```

**Expected output:**
- MySQL version information
- List of databases (including mysql, information_schema, performance_schema, sys)

### Test MySQL Service Status

**Windows (Command Prompt as Administrator):**
```cmd
sc query MySQL80
```

**Linux:**
```bash
sudo systemctl status mysql
```
or
```bash
sudo systemctl status mysqld
```

---

## Common Installation Issues and Solutions

### Issue 1: Port 3306 Already in Use (Windows)
**Solution:**
- Check if another MySQL instance is running
- Stop the existing service: `net stop MySQL80`
- Or configure MySQL to use a different port during installation

### Issue 2: Can't Connect to MySQL Server (Linux)
**Solution:**
- Check if MySQL is running: `sudo systemctl status mysql`
- Start MySQL if stopped: `sudo systemctl start mysql`
- Check firewall settings: `sudo ufw allow 3306/tcp`

### Issue 3: Access Denied for Root User
**Solution:**
- Reset root password using `mysql_secure_installation`
- Or use: `sudo mysql` (on Linux) to access without password, then reset

### Issue 4: Missing Dependencies (Linux)
**Solution:**
- Update package lists: `sudo apt update`
- Install missing dependencies: `sudo apt install -f`

---

## Next Steps

After successful installation:
1. **Install MySQL Workbench** (if not already installed) for a graphical interface
2. **Create practice databases** following the course modules
3. **Configure your development environment** with your preferred SQL client
4. **Start learning** with Module 01 in the lessons folder

---

## Additional Resources

- **Official MySQL Documentation**: https://dev.mysql.com/doc/
- **MySQL Workbench User Guide**: https://dev.mysql.com/doc/workbench/en/
- **MySQL Tutorial**: https://dev.mysql.com/doc/mysql-tutorial-excerpt/8.0/en/
- **MySQL Community Support**: https://forums.mysql.com/

---

**Note**: Always use strong passwords and keep your MySQL installation updated with the latest security patches.

Good luck with your SQL learning journey! 🚀
