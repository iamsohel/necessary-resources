### Odoo community deploy in AWS EC2

2️⃣ Connect via SSH

`ssh -i your-key.pem ubuntu@your-ec2-public-ip`

3️⃣ Update and Install Required Packages

`sudo apt update && sudo apt upgrade -y
sudo apt install git python3-pip build-essential wget python3-dev python3-venv \
libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less \
libjpeg-dev libpq-dev libxml2-dev npm -y`

4️⃣ Install PostgreSQL

`sudo apt install postgresql -y
sudo systemctl enable postgresql
sudo systemctl start postgresql`


Create the Odoo database user:

`sudo -u postgres createuser -s odoo
sudo -u postgres psql -c "ALTER USER odoo WITH PASSWORD 'odoo_db_password';"`

5️⃣ Create Odoo User & Directory

`sudo useradd -m -d /opt/odoo -U -r -s /bin/bash odoo
sudo su - odoo`

6️⃣ Install Odoo 18

`git clone https://www.github.com/odoo/odoo --branch 18.0 --depth 1
cd odoo
python3 -m venv venv
source venv/bin/activate
pip install wheel
pip install -r requirements.txt`

Exit from odoo user:

`exit`

7️⃣ Create Odoo Configuration File

sudo nano /etc/odoo.conf

Paste:

    `ini
        
      [options]
      ; Database
      db_host = False
      db_port = False
      db_user = odoo
      db_password = odoo_db_password
      
    ; Paths
    addons_path = /opt/odoo/odoo/addons
    admin_passwd = master_password
    
    ; Server
    xmlrpc_port = 8069
    logfile = /var/log/odoo/odoo.log
  `
  
8️⃣ Create Log Directory

`sudo mkdir -p /var/log/odoo
sudo chown odoo: /var/log/odoo`

9️⃣ Create Odoo Systemd Service

sudo nano /etc/systemd/system/odoo.service


    `ini
    
    [Unit]
    Description=Odoo
    Documentation=http://www.odoo.com
    After=network.target
    
    [Service]
    Type=simple
    User=odoo
    ExecStart=/opt/odoo/odoo/venv/bin/python3 /opt/odoo/odoo/odoo-bin -c /etc/odoo.conf
    Restart=always
    
    [Install]
    WantedBy=multi-user.target`


🔟 Start and Enable Odoo

`sudo systemctl daemon-reload
sudo systemctl enable --now odoo
sudo systemctl status odoo`


1️⃣1️⃣ Access Odoo
Open:
add custom port in sucurity group 8069
http://<EC2_PUBLIC_IP>:8069
You’ll see the Odoo setup screen.

