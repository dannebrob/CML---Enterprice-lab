This document describes the installation of NetBox on Ubuntu 22.04 LTS. The installation process is based on the official NetBox documentation and includes additional steps to ensure a smooth setup. The purpose of installing NetBox is to have the ability to automate a lot of the network management tasks and to have a central repository for all the network devices and their configurations.

#Installation of NetBox on Ubuntu 22.04 LTS

# Prerequisites
Install the necessary dependencies for NetBox:

```bash
sudo apt update

# Install Postgres
sudo apt install -y postgresql
psql -V 

## Create the database, with user netbox and pass SUPER_SECRET_PASSWORD
CREATE DATABASE netbox;
CREATE USER netbox WITH PASSWORD 'SUPER_SECRET_PASSWORD';
ALTER DATABASE netbox OWNER TO netbox;
-- the next two commands are needed on PostgreSQL 15 and later
\connect netbox;
GRANT CREATE ON SCHEMA public TO netbox;

## confirm working database
psql --username netbox --password --host localhost netbox
# confirm network
\conninfo 

## exit 
\q 

# Install Redis
sudo apt install -y redis-server

## confirm working redis
redis-server -v

## test, expected return: PONG
redis-cli ping

# Check if python is installed
python3 -V

# install python virtual enviroment
sudo apt install python3.12-venv

```

# Install Netbox

```bash
#Note for future: Check if latest version is 4.5.5, if true: replace
sudo wget https://github.com/netbox-community/netbox/archive/refs/tags/v4.5.5.tar.gz
sudo tar -xzf v4.5.5.tar.gz -C /opt
sudo ln -s /opt/netbox-4.5.5/ /opt/netbox


sudo adduser --system --group netbox
sudo chown --recursive netbox /opt/netbox/netbox/media/
sudo chown --recursive netbox /opt/netbox/netbox/reports/
sudo chown --recursive netbox /opt/netbox/netbox/scripts/

cd /opt/netbox/netbox/netbox/
sudo cp configuration_example.py configuration.py
sudo nano configuration.py

# Add/edit these in the config file

ALLOWED_HOSTS = ['*']

API_TOKEN_PEPPERS = {
    # DO NOT USE THIS EXAMPLE PEPPER IN PRODUCTION
    1: 'kp7ht*76fiQAhUi5dHfASLlYUE_S^gI^(7J^K5M!LfoH@vl&b_' 
    }
DATABASES = {
    'default': {
        'NAME': 'netbox',               # Database name
        'USER': 'netbox',               # PostgreSQL username
        'PASSWORD': 'SUPER_SECRET_PASSWORD', # PostgreSQL password
        'HOST': 'localhost',            # Database server
        'PORT': '',                     # Database port (leave blank for default)
        'CONN_MAX_AGE': 300,            # Max database connection age (seconds)
    }
}
REDIS = {
    'tasks': {
        'HOST': 'localhost',      # Redis server
        'PORT': 6379,             # Redis port
        'PASSWORD': '',           # Redis password (optional)
        'DATABASE': 0,            # Database ID
        'SSL': False,             # Use SSL (optional)
    },
    'caching': {
        'HOST': 'localhost',
        'PORT': 6379,
        'PASSWORD': '',
        'DATABASE': 1,            # Unique ID for second database
        'SSL': False,
    }
}

# Generate a key from python3 ../generate_secret_key.py
SECRET_KEY = 'J0n_9PlRF)=RJ&dr@mJqrnhQ_kURiUuH-THISISNOTTHEREALKEY'
```

# Upgrade Netbox


## resources- [NetBox Documentation](https://netboxlabs.com/docs/netbox/installation/)
