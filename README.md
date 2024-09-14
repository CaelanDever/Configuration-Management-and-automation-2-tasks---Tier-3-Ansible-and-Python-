# Configuration-Management-and-automation-2-tasks---Tier-3-Ansible-and-Python

# Ansible =============

### Task: Ansible Playbook for Web Server Environment Setup

#### Description:
Write an Ansible playbook to automate the setup of a web server environment using Nginx and PHP on multiple CentOS servers. The playbook should install and configure Nginx as the web server and PHP for dynamic content processing. Additionally, configure a basic PHP info page for testing purposes. Test the playbook on a test environment to verify its functionality.


Install Python 3.12 manually and then install Ansible
Install Python 3.12 manually by building it from source:


sudo dnf groupinstall "Development Tools"
sudo dnf install wget bzip2-devel libffi-devel zlib-devel xz-devel openssl-devel
wget https://www.python.org/ftp/python/3.12.0/Python-3.12.0.tgz
tar -xf Python-3.12.0.tgz
cd Python-3.12.0
./configure --enable-optimizations
sudo make altinstall
Install Ansible after installing Python 3.12:

Update alternatives to use Python 3.12 as the default:


sudo alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.12 1
Then, try installing Ansible again:

sudo dnf install ansible
Here's a complete Ansible playbook for automating the setup of a web server environment using Nginx and PHP on CentOS servers:

<img width="455" alt="kkk" src="https://github.com/user-attachments/assets/88df9d8e-ff84-469f-8cdc-ef338d1bfc64">

# Directory Structure
For ease of use, let's assume your playbook files will be organized like this:

├── playbook.yml
├── templates
│   └── nginx.conf.j2
└── files
    └── info.php

Step-by-Step Ansible Playbook

# 1. playbook.yml
This is the main playbook file that includes all the tasks.

---
- name: Setup Nginx and PHP Web Server Environment
  hosts: web_servers
  become: yes
  vars:
    php_packages:
      - php
      - php-fpm
      - php-mysql
      - php-cli
      - php-common
      - php-xml
      - php-json
      - php-mbstring
    firewalld_services:
      - http

  tasks:
    - name: Ensure Nginx and PHP-FPM are installed
      yum:
        name:
          - nginx
          - "{{ php_packages }}"
        state: present
      notify:
        - restart nginx
        - restart php-fpm

    - name: Create Nginx configuration file from template
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify:
        - restart nginx

    - name: Deploy PHP info page
      copy:
        src: files/info.php
        dest: /usr/share/nginx/html/info.php
        mode: '0644'

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Start and enable PHP-FPM
      service:
        name: php-fpm
        state: started
        enabled: true

    - name: Configure firewalld to allow HTTP traffic
      firewalld:
        service: "{{ firewalld_services }}"
        permanent: yes
        state: enabled
        immediate: yes

    - name: Ensure firewalld is running and enabled on boot
      service:
        name: firewalld
        state: started
        enabled: true

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted

    - name: restart php-fpm
      service:
        name: php-fpm
        state: restarted

# 2. nginx.conf.j2
This template configures Nginx to serve PHP files.

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;
    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;

        location / {
            index index.php index.html index.htm;
        }

        location ~ \.php$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }
}
3. info.php
This is a simple PHP info page to verify the PHP setup.

php
Copy code
<?php
phpinfo();
?>

# Breakdown of the Playbook
Install Nginx and PHP:

The yum module installs Nginx and the PHP packages specified in the variable php_packages.

# Configure Nginx:

The template module copies the nginx.conf.j2 template to /etc/nginx/nginx.conf and triggers the Nginx restart handler.
Deploy PHP Info Page:

The copy module places the info.php file in Nginx's web root (/usr/share/nginx/html/) for testing PHP functionality.
Start and Enable Services:

The service module ensures Nginx and PHP-FPM are started and enabled on boot.

# Configure Firewall:

The firewalld module enables HTTP (port 80) traffic on the system's firewall.
Error Handling and Notifications:

The playbook uses handlers to restart Nginx and PHP-FPM when their configuration changes.
Testing the Playbook

After creating the playbook and its associated files, you can test it by running the playbook on your CentOS servers. Ensure you have a group [web_servers] defined in your hosts inventory file.

Sample Inventory File (hosts)

[web_servers]
server1.example.com
server2.example.com

# Run the Playbook
To execute the playbook, use the following command:

ansible-playbook -i hosts playbook.yml

# Completion Criteria

Nginx and PHP Installed: The playbook installs and configures Nginx and PHP (with required extensions).

PHP Info Page: A PHP info page is deployed and accessible via http://<server_ip>/info.php.

Service Configuration: Nginx and PHP-FPM services are enabled and running.

Firewall Configured: Firewalld is configured to allow HTTP traffic.

Error Handling: If a package installation fails or service doesn't start, the playbook will gracefully handle those errors by retrying or stopping execution with an appropriate message.

This playbook should fulfill the task of setting up a web server environment across multiple CentOS servers.


# Python ==============

# Task: Python Script for Automated File Backup

Here is the Python script with detailed explanations for each step to ensure it meets all the criteria for automating backups on a CentOS server. I will go over each section of the script, explaining how it handles the various requirements like recursive directory traversal, error handling, and the creation of compressed .tar.gz archives.

# Python Script for Automated File Backup with Detailed Explanations

Make the directories for the data and for backup 

<img width="384" alt="s2" src="https://github.com/user-attachments/assets/f3374ebd-6c8d-4f72-b237-3242f9aec8d0">

Detailed Step-by-Step Explanation

# Step 1: Define Files and Directories to Back Up
In the files_to_backup list, we specify both files and directories that need to be backed up. This can include paths to any files and folders. For example: Pic is shown after step 2

# These are files that I made to be tested with the script:

<img width="331" alt="ee" src="https://github.com/user-attachments/assets/bcf0ab29-7c13-4847-be14-a029d16e5345">


This will enable the script to traverse directories recursively and back up all files within those directories.

# Step 2: Define the Backup Location

The backup_location variable defines where the backup will be stored. It is a directory where the script will create timestamped subdirectories for each backup. You can set this path based on your CentOS server’s directory structure:

<img width="450" alt="211" src="https://github.com/user-attachments/assets/90be2ae6-dc54-4ba2-a2c9-07f04eaa430b">

# Step 3: Generate a Timestamped Folder for Backups
The script uses the datetime module to generate a unique timestamp for each backup:


<img width="447" alt="124" src="https://github.com/user-attachments/assets/022a99a0-5b46-4e42-b9da-31d966601966">

This ensures that each backup is stored in its own folder, and no backups overwrite previous ones.

# Step 4: Create a .tar.gz Archive
To compress files and directories, the script uses the tarfile module. It creates a .tar.gz archive with the following:

with tarfile.open(backup_file, 'w:gz') as tar:
    tar.add(item, arcname=os.path.basename(item))
This creates a tar archive and adds each file or directory to it while preserving the original directory structure. The compression saves space on the backup disk.

Step 5: Error Handling for Robustness
The script includes several error handling mechanisms:

PermissionError: This handles issues where the script doesn’t have permission to access certain files or directories.
OSError: This captures operating system-level errors such as missing directories, disk space issues, etc.

General Exceptions: Any other unforeseen errors are caught and reported.
For example:

<img width="451" alt="512" src="https://github.com/user-attachments/assets/ba583289-7f93-49cd-ac7e-febe3bde4eaa">

This ensures the script can gracefully handle any errors and continue functioning, either by skipping problematic files or alerting the user to fix the issue.

# Step 6: Check and Create Backup Location
Before creating the backup, the script checks if the specified backup_location exists. If it doesn’t, the script tries to create it:

if not os.path.exists(backup_location):
    os.makedirs(backup_location)

    If this operation fails due to permissions or other reasons, an error is reported, and the backup process stops.

# Step 7: Main Function to Trigger Backup
The main() function orchestrates the backup by first checking the backup location and then calling the create_backup() function to perform the actual backup.

# Additional Considerations

Recursive Traversal: The tar.add() method is inherently recursive when adding directories, so all files within the specified directories will be included.

Testing: Before deployment, ensure the script has access to all the files, directories, and the backup location. Run tests on your CentOS server using the command:

<img width="397" alt="122" src="https://github.com/user-attachments/assets/77f37d99-4501-4e18-aff6-2a0bf376a570">

How to Automate the Backup (Optional)
To automate this backup script, you can set up a cron job on your CentOS server. For example:

Open the cron table for the root user

sudo crontab -e
Add the following line to schedule the script to run daily at 2 AM:
0 2 * * * /usr/bin/python3 /path/to/backup_script.py

This cron job will run the script daily, ensuring your backups are performed automatically.

Let me know if you'd like further customization or clarification!clear

import os
import tarfile
import shutil
from datetime import datetime

# 1. Define files and directories to back up
# This list includes both files and directories you wish to back up.
files_to_backup = [
    '/path/to/directory1',  # Example directory to back up
    '/path/to/directory2',  # Another directory to back up
    '/path/to/file1.txt'    # An individual file to back up
]

# 2. Define the backup location
# This is the directory where the backup archives will be stored.
backup_location = '/path/to/backup_location'

def create_backup(backup_items, backup_dir):
    """
    This function handles the creation of a backup by compressing the 
    specified files and directories into a timestamped .tar.gz archive.
    
    Parameters:
    backup_items: list of paths (files and directories) to be backed up
    backup_dir: the location where the backup will be stored
    
    """
    try:
        # 3. Generate a timestamped folder to store the backup
        # The datetime module is used to generate a unique timestamp for each backup.
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_folder = os.path.join(backup_dir, f'backup_{timestamp}')
        
        # Create the backup folder if it doesn't already exist
        os.makedirs(backup_folder, exist_ok=True)
        
        # 4. Define the name of the compressed archive (tar.gz)
        # The backup will be compressed into a .tar.gz archive file.
        backup_file = os.path.join(backup_folder, f'backup_{timestamp}.tar.gz')

        # 5. Create the .tar.gz archive and add files/directories to it
        # The 'w:gz' mode is used to create a compressed tar.gz archive.
        with tarfile.open(backup_file, 'w:gz') as tar:
            for item in backup_items:
                # Check if the file or directory exists before adding to the backup
                if os.path.exists(item):
                    # Add the file/directory to the tar archive, preserving its structure
                    tar.add(item, arcname=os.path.basename(item))
                else:
                    print(f'Warning: {item} does not exist and will be skipped.')
        
        print(f'Backup created successfully: {backup_file}')
    except PermissionError:
        # Handle permission errors, which may occur if the script does not have access to the file
        print("Error: Permission denied. Please check your file and directory permissions.")
    except OSError as e:
        # Catch and display any other OS-related errors (like missing directories or disk issues)
        print(f"Error: OS error occurred: {e}")
    except Exception as e:
        # Catch any other general errors to ensure robustness
        print(f"Error: An unexpected error occurred: {e}")

def main():
    """
    This is the main function that triggers the backup process. It checks the 
    backup location, ensures it exists, and then calls the create_backup function.
    """
    # 6. Check if the backup location exists and create it if necessary
    if not os.path.exists(backup_location):
        try:
            os.makedirs(backup_location)
        except PermissionError:
            print("Error: Could not create backup location due to permission issues.")
            return
        except Exception as e:
            print(f"Error: Could not create backup location: {e}")
            return
    
    # 7. Start the backup process
    create_backup(files_to_backup, backup_location)

if __name__ == "__main__":
    main()



# RESULTS:

Backup location before script is ran: None of the files I made above are there yet:


<img width="437" alt="b4" src="https://github.com/user-attachments/assets/1333ea3e-7b7c-4b60-97b9-b5d49f367192">

After the script is ran:


<img width="480" alt="a4" src="https://github.com/user-attachments/assets/eaa15053-51dd-4092-915b-ab4383a507b4">



