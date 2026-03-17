# Ansible Real-Time Project

**Project Goal**  
Automate MySQL database backup on an AWS EC2 Ubuntu instance and store the backup in an S3 bucket using Ansible.

#### Steps Involved

1. **Install MySQL on EC2 Ubuntu Machine**  
   Set up and configure MySQL database server on a Ubuntu-based EC2 instance.
   Installing MySQL 8.0 on Ubuntu (e.g., Ubuntu 20.04, 22.04, or 24.04)

These commands install the official MySQL Community Server 8.0 from the MySQL APT repository.

#### Step-by-step Installation

    1. **Update package index**
        ```bash
        sudo apt update
    2. **Install required tools (if not already present)**
        sudo apt install -y wget gnupg lsb-release
    3. **Download and add the MySQL APT repository**
        wget https://repo.mysql.com/mysql-apt-config_0.8.32-1_all.deb
        sudo dpkg -i mysql-apt-config_0.8.32-1_all.deb
        
        **Note**
        During installation of the .deb package, a configuration window will appear.
        Select MySQL Server & Cluster → mysql-8.0 → OK
    4. **Update package index again after adding the repo** 
        sudo apt update
    5. **Install MySQL Server 8.0**
        sudo apt install -y mysql-server
    6. **(Optional) Secure the installation (highly recommended)**
        sudo mysql_secure_installation
    7. **Check MySQL service status;Start / Restart / Enable MySQL service (if needed)**
        sudo systemctl status mysql
        sudo systemctl start mysql
        sudo systemctl enable mysql
        sudo systemctl restart mysql
    8. **Verify MySQL version** 
        mysql --version
        # or
        mysql -u root -p -e "SELECT VERSION();"


2. **Create Database and Table on MySQL**  
   Create a sample database and at least one table to demonstrate backup functionality.
    
    # Create Database and Table in MySQL on Ubuntu

    After successfully installing MySQL on your Ubuntu machine, follow the steps below to create a database and table.

    ### Step-by-Step Commands

    1. **Login to MySQL Server**
        ```bash
         sudo mysql
    Note: On fresh Ubuntu installations with MySQL 8.0, you can log in as root without a password using sudo mysql.
    If you have set a root password, use: mysql -u root -p
    2. **Show existing databases**
        SHOW DATABASES;
    3. **Create a new database** 
        CREATE DATABASE Morise;
    4. **Switch to the newly created database**
        USE Morise;
    5. **Create a table**
        CREATE TABLE tblEmployee1 (
        Employee_first_name VARCHAR(500) NOT NULL,
        Employee_last_name  VARCHAR(500) NOT NULL
        );
    6. **Insert sample data**
        INSERT INTO tblEmployee1 (Employee_first_name, Employee_last_name)
        VALUES ('Obeke', 'Godfrey');
    7. **Verify the data**
        SELECT * FROM tblEmployee1;
    8. **Exit MySQL shell**
        EXIT;
        
3. **Install AWS CLI on EC2 Instance and Configure the Credentials**  
   Install the AWS Command Line Interface and configure it with appropriate IAM credentials (preferably using an EC2 instance role).

    1. **Update package list and install unzip**
        sudo apt update && sudo apt install -y unzip

    2. **Download AWS CLI v2**
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

    3. **Unzip and install**
        unzip awscliv2.zip
        sudo ./aws/install

    4. **Verify installation**
        aws --version

    5. **Configure AWS credentials**
        aws configure


4. **Install Ansible**  
   Install Ansible on the EC2 instance (or on a separate control node, depending on your setup).
   
    1. **Update system and install dependencies**
        sudo apt update && sudo apt install -y software-properties-common

    2. **Add Ansible official PPA**
        sudo add-apt-repository --yes --update ppa:ansible/ansible

    3. **Install Ansible**
        sudo apt install -y ansible

    4. **Verify installation**
        ansible --version

    5. **Write the Ansible Playbook**  
      Create an Ansible playbook that:  
       - Takes a MySQL database backup (using `mysqldump`)  
       - Optionally compresses the backup file  
       - Uploads the backup to an AWS S3 bucket

    Ansible Playbook - MySQL Backup to S3 Bucket

    Create a playbook file to take a MySQL backup and upload it to AWS S3.

    1. **Create the Playbook**
    
      **Run the following command to create the playbook file:**
        vi main.yaml

    2. **Paste the following content into main.yaml:**   
        ---
        - name: MySQL Backup and Upload to S3
          hosts: localhost
          become: yes
          tasks:
            - name: Take MySQL backup using mysqldump
              shell: >
                mysqldump -u root 
                --password="yourpassword" 
                Morise tblEmployee1 > /tmp/Morise_backup.sql
              args:
                creates: /tmp/Morise_backup.sql
              register: backup_result

            - name: Display backup status
              debug:
                msg: "MySQL backup completed successfully at /tmp/Morise_backup.sql"

            - name: Upload MySQL backup to S3 bucket
              shell: /usr/local/bin/aws s3 cp /tmp/Morise_backup.sql s3://ansible-mysql-bucket/
              register: upload_result

            - name: Display upload status
              debug:
                msg: "Backup successfully uploaded to S3 bucket: ansible-mysql-bucket"
                
    3. **Run the Playbook**    
          ansible-playbook main.yaml
