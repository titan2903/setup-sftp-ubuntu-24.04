# Install and Configure SFTP Server Ubuntu 24.04

## 1. **Updating Packages and Installing OpenSSH Server**
   The first command updates the package list and installs the `openssh-server` required to run the SSH server.

   ```
   sudo apt update
   ```
   - **Explanation**: This command updates the package repository list on your system.

   ```
   sudo apt install openssh-server -y
   ```
   - **Explanation**: Installs the `openssh-server` package, which provides the SSH service required to connect to the server over SSH protocol (including SFTP).

## 2. **Starting and Enabling the SSH Service**
   After installing `openssh-server`, you need to start and enable the SSH service to run on system startup.

   ```
   sudo systemctl start ssh
   ```
   - **Explanation**: Starts the SSH service immediately.

   ```
   sudo systemctl enable ssh
   ```
   - **Explanation**: Enables the SSH service to start automatically on boot.

## 3. **Checking the SSH Service Status**
   Verify that the SSH service is running correctly.

   ```
   sudo systemctl status ssh
   ```
   - **Explanation**: This command shows the status of the SSH service. It helps confirm whether SSH is running without any issues.

## 4. **Creating a Group and User for SFTP**
   - Create a special group `sftp_users` and a user `sftpuser` that will be restricted to using SFTP only.

   ```
   sudo groupadd sftp_users
   ```
   - **Explanation**: Creates the `sftp_users` group, which will be used to restrict access to users who are part of this group.

   ```
   sudo useradd -m -G sftp_users -s /usr/sbin/nologin sftpuser
   ```
   - **Explanation**: Creates a new user `sftpuser`, adds it to the `sftp_users` group, and ensures the user cannot log in via SSH shell by setting the shell to `/usr/sbin/nologin`.

## 5. **Adding Configuration to the `sshd_config` File**
   This step involves editing the SSH configuration file to specify access restrictions for the `sftp_users` group and allow only SFTP access, not shell access.

   - Open the SSH configuration file (`sshd_config`) for editing:
     ```
     sudo nano /etc/ssh/sshd_config
     ```

   - **Add the following configuration at the end of the file**:
     ```
     Match Group sftp_users
     ChrootDirectory %h
     ForceCommand internal-sftp
     AllowTcpForwarding no
     X11Forwarding no
     ```
     - **Explanation**:
       - `Match Group sftp_users`: This section applies the settings to the `sftp_users` group.
       - `ChrootDirectory %h`: Restricts users to their home directories so they cannot access other parts of the system.
       - `ForceCommand internal-sftp`: Forces users to use SFTP and prevents them from executing other commands, such as SSH shell.
       - `AllowTcpForwarding no`: Disables TCP forwarding to enhance security.
       - `X11Forwarding no`: Disables X11 forwarding, preventing remote graphical access.

## 6. **Restarting the SSH Service**
   After modifying the configuration file, restart the SSH service to apply the changes.

   ```
   sudo systemctl restart ssh
   ```
   - **Explanation**: Restarts the SSH service to apply the changes made to the configuration file.

## 7. **Setting Up Directories and Permissions for the SFTP User**
   - Adjust permissions on the SFTP user's home directory and create an `uploads` directory where files can be uploaded via SFTP.

   ```
   sudo chown root:root /home/sftpuser
   ```
   - **Explanation**: Changes the ownership of the `sftpuser` home directory to root. This is required because the `ChrootDirectory` policy mandates that the home directory must be owned by root.

   ```
   sudo chmod 755 /home/sftpuser
   ```
   - **Explanation**: Grants read, write, and execute permissions to the owner (root) and read and execute permissions to others.

   ```
   sudo mkdir /home/sftpuser/uploads
   ```
   - **Explanation**: Creates an `uploads` directory inside the `sftpuser` home directory where files can be uploaded via SFTP.

   ```
   sudo chown sftpuser:sftp_users /home/sftpuser/uploads
   ```
   - **Explanation**: Changes the ownership of the `uploads` directory to the `sftpuser` and `sftp_users` group, allowing the user to upload files there.

---

By following these steps, you have successfully set up SFTP on Ubuntu 24.04, with the `sftpuser` being restricted to SFTP access only and allowed to upload files to the designated `uploads` directory. Be sure to check each step carefully to avoid configuration errors.