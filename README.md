# linux-file-auto-owners
You can modify the script to automatically apply it to all users' home directories. You can achieve this by looping through all the directories in `/home/`.

Here’s how you can modify the script:

### 1. **Install inotify-tools (if not already installed)**

```sh
sudo apt update
sudo apt install inotify-tools  # For Debian/Ubuntu

# OR

sudo yum install inotify-tools  # For CentOS/Red Hat
```

### 2. **Create a Script to Monitor All Users' Home Directories**

Create a new file, for example, `monitor_all_users.sh`, and open it in a text editor.

```sh
nano monitor_all_users.sh
```

Add the following content to the `monitor_all_users.sh` file.

```sh
#!/bin/bash

inotifywait -m -r -e create,modify,move,delete --format '%w%f' /home/ | while read file
do
    username=$(echo "$file" | cut -d '/' -f3)
    if [ -n "$username" ]; then
        chown $username:$username "$file"
        chmod 750 "$file"   # Modifying permission here
    fi
done
```

In this script:
- The `inotifywait` command is used to monitor all subdirectories in `/home/` recursively for file changes.
- The `cut` command extracts the username from the file path. This assumes a standard path structure, like `/home/username/...`.
- The `chown` command changes the file’s owner to the extracted username.

### 3. **Make the Script Executable**

```sh
chmod +x monitor_all_users.sh
```

### 4. **Run the Script in the Background**

```sh
nohup ./monitor_all_users.sh &
```

### Notes:

1. This script assumes that each user's home directory is under `/home/username`. If there's a different structure, you need to modify the script accordingly.

2. As always, be cautious with recursive options and ensure you are not unintentionally giving access to sensitive files or system directories.

3. This script also needs to be run with root privileges to change the ownership of files. Be very careful when running scripts as root, and ensure your script is secure and tested to avoid unintended consequences.

4. This is a basic implementation for illustrative purposes. Depending on your exact requirements and constraints, you might need to modify or expand it, adding error handling, logging, and other necessary features for a production environment.

5. Certainly! You can create a systemd service to manage the script. Follow the steps below to create a systemd service for the script.

### 1. **Move the Script to an Appropriate Directory**

Move the script to a standard directory for executables, for example, `/usr/local/bin/`.

```sh
sudo mv monitor_all_users.sh /usr/local/bin/
```

### 2. **Create a Systemd Service File**

Create a new systemd service file using a text editor. Here, I'll use `nano`.

```sh
sudo nano /etc/systemd/system/file-monitor.service
```

### 3. **Add the Following Content to the Service File**

Replace `/usr/local/bin/monitor_all_users.sh` with the path to your script if it's different.

```sh
[Unit]
Description=File Monitor to Auto Change Owner
After=network.target

[Service]
ExecStart=/usr/local/bin/monitor_all_users.sh
Restart=always
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

### 4. **Reload Systemd and Start the Service**

Now reload the systemd daemon and enable/start the service.

```sh
sudo systemctl daemon-reload
sudo systemctl enable file-monitor
sudo systemctl start file-monitor
```

### 5. **Check the Status of the Service**

You can check the status of your service to make sure it's running correctly.

```sh
sudo systemctl status file-monitor
```

### Explanation:

- The `[Unit]` section contains the metadata and dependencies, including a description and the conditions under which the service should be started.

- The `[Service]` section provides the start-up behavior of the service and specifies how to start the service.

- The `ExecStart` parameter is used to specify the command to start the service.

- The `Restart=always` parameter ensures the service is restarted if it fails.

- The `[Install]` section allows the service to be enabled to start at boot.

- `WantedBy=multi-user.target` ensures the service starts when the system is booted into a multi-user mode.

Now, your script is running as a service, and it will start automatically with the system boot. You can control it using `systemctl start`, `systemctl stop`, and `systemctl restart` commands followed by the service name.
