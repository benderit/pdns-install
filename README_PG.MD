# Postgresql Installation for PowerDNS

## Setting up the Basics 

First we’ll need to install some dependencies and set the PowerDNS Repositories. We’ll also create a directory to put our generated credentials and files in.

Login to superuser
```bash
su
```

```bash
# Install basic dependencies
apt-get install software-properties-common gnupg2 lsb-release sudo curl -y
usermod -aG sudo $USER
cat << EOF | tee /etc/sudoers.d/$USER
$USER ALL=(ALL:ALL) NOPASSWD: ALL
EOF
exit
```


### Postgresql (PGSQL) 

```bash
# Set some variables for repository setup
export systemReleaseVersion=$(lsb_release -cs)
export osLabel=$(lsb_release -sa 2>/dev/null|head -n 1|tr '[:upper:]' '[:lower:]')
```

```bash
# Add PostgreSQL Public Key
wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor --output /etc/apt/trusted.gpg.d/postgresql.gpg

# Add PostgreSQL Repository
echo "deb http://apt.postgresql.org/pub/repos/apt ${systemReleaseVersion}-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list

# Update the package lists:
sudo apt-get update && sudo apt-get upgrade

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-12' or similar instead of 'postgresql':
sudo apt-get -y install postgresql

echo "listen_addresses = '0.0.0.0'" | sudo tee -a /etc/postgresql/*/main/postgresql.conf
echo "host    all             all             192.168.22.0/24         scram-sha-256" | sudo tee -a /etc/postgresql/*/main/pg_hba.conf

# Enable PostgreSQL Service
sudo systemctl enable --now postgresql

# Start PostgreSQL Service
sudo systemctl restart postgresql
```

Don’t forget to set your local unix socket connection in the pg_hba.conf file to md5 or scram-sha-256 instead of peer


## Generating the Database 

Once the DBMS is installed you’ll need to generate credentials for PowerDNS and create it’s DB Schema.

```bash
export workpath="/opt/pdns_install"
export systemReleaseVersion=$(lsb_release -cs)
export osLabel=$(lsb_release -sa 2>/dev/null|head -n 1|tr '[:upper:]' '[:lower:]')
export pdns_db="pdns"
export pdns_db_user="pdnsadmin"
export pdns_pwd="$(tr -dc A-Za-z0-9 </dev/urandom | head -c 13 ; echo '')"
export pdnsadmin_salt="$(tr -dc A-Za-z0-9 </dev/urandom | head -c 32 ; echo '')"
export pdns_apikey="$(tr -dc A-Za-z0-9 </dev/urandom | head -c 32 ; echo '')"
export db_type="pgsql"

echo "export pdns_db=$pdns_db" > "db_credentials"
echo "export pdns_db_user=$pdns_db_user" >> "db_credentials"
echo "export pdns_pwd=$pdns_pwd" >> "db_credentials"
echo "export pdnsadmin_salt=$pdnsadmin_salt" >> "db_credentials"
echo "export pdns_apikey=$pdns_apikey" >> "db_credentials"
echo "export workpath=$workpath" >> "db_credentials"
echo 'export systemReleaseVersion=$(lsb_release -cs)' >> "db_credentials"
echo 'export osLabel=$(lsb_release -sa 2>/dev/null|head -n 1|tr '[:upper:]' '[:lower:]')' >> "db_credentials"
echo "export db_type=$db_type" >> "db_credentials"
cat db_credentials
```

```bash
cat << EOF | sudo tee "pdns-createdb-pg.sql"
-- PowerDNS PGSQL Create DB File
CREATE USER $pdns_db_user WITH ENCRYPTED PASSWORD '$pdns_pwd';
CREATE DATABASE $pdns_db OWNER $pdns_db_user;
GRANT ALL PRIVILEGES ON DATABASE $pdns_db TO $pdns_db_user;
EOF

sudo -u postgres psql < "pdns-createdb-pg.sql"
```

```bash
# Temporary install PowerDNS Authoritative Server for setup schema
sudo apt-get install pdns-server pdns-backend-pgsql -y && \
sudo systemctl stop pdns

export PGPASSWORD="$pdns_pwd"
psql -U $pdns_db_user -h 127.0.0.1 -d $pdns_db < "/usr/share/pdns-backend-$db_type/schema/schema.$db_type.sql"
unset PGPASSWORD

sudo apt-get purge pdns-server pdns-backend-pgsql -y
```
