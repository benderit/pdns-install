## Setting up the Basics 

First we’ll need to install some dependencies and set the PowerDNS Repositories. We’ll also create a directory to put our generated credentials and files in.

Login to superuser
```bash
su
```
### Install prerequisite packages 
```bash
apt-get update && \
apt-get install software-properties-common gnupg2 lsb-release sudo curl -y
```

### Set-up sudo for user
```bash
/usr/sbin/usermod -aG sudo $USER
cat << EOF | tee /etc/sudoers.d/$USER
$USER ALL=(ALL:ALL) NOPASSWD: ALL
EOF
```

### Adding the Source Repositories 

```bash
export systemReleaseVersion=$(lsb_release -cs)
export osLabel=$(lsb_release -sa 2>/dev/null|head -n 1|tr '[:upper:]' '[:lower:]')

# PowerDNS Authoritative Repository
repo_pdnsAuth="deb [arch=amd64] http://repo.powerdns.com/${osLabel} ${systemReleaseVersion}-auth-48 main"
echo $repo_pdnsAuth | sudo tee "/etc/apt/sources.list.d/pdns-auth.list"

# PowerDNS Recursor Repository
repo_pdnsRec="deb [arch=amd64] http://repo.powerdns.com/${osLabel} ${systemReleaseVersion}-rec-48 main"
echo $repo_pdnsRec | sudo tee "/etc/apt/sources.list.d/pdns-rec.list"

# PowerDNS DNS Dist Repository
repo_dnsdist="deb [arch=amd64] http://repo.powerdns.com/${osLabel} ${systemReleaseVersion}-dnsdist-18 main"
echo $repo_dnsdist | sudo tee "/etc/apt/sources.list.d/dnsdist.list"

# Get the current key from the PowerDNS Repository, this could be outdated

# Master Branch
wget -qO- https://repo.powerdns.com/FD380FBB-pub.asc | sudo gpg --dearmor --output /etc/apt/trusted.gpg.d/pdns_master.gpg

# Stable Branch
wget -qO- https://repo.powerdns.com/FD380FBB-pub.asc | sudo gpg --dearmor --output /etc/apt/trusted.gpg.d/pdns_stable.gpg

cat << EOF | sudo tee /etc/apt/preferences.d/pdns
Package: pdns-*
Pin: origin repo.powerdns.com
Pin-Priority: 600
EOF
```
