## Setting up the Basics 

First we’ll need to install some dependencies and set the PowerDNS Repositories. We’ll also create a directory to put our generated credentials and files in.

Login to superuser
```bash
su
```

```bash
# Install basic dependencies
apt-get update && \
apt-get install software-properties-common gnupg2 lsb-release sudo curl -y
usermod -aG sudo $USER
cat << EOF | tee /etc/sudoers.d/$USER
$USER ALL=(ALL:ALL) NOPASSWD: ALL
EOF
```

```bash
# Get the current key from the PowerDNS Repository, this could be outdated
# Master Branch
wget -qO- https://repo.powerdns.com/FD380FBB-pub.asc | sudo gpg --dearmor --output /etc/apt/trusted.gpg.d/pdns_master.gpg
# Stable Branch
wget -qO- https://repo.powerdns.com/FD380FBB-pub.asc | sudo gpg --dearmor --output /etc/apt/trusted.gpg.d/pdns_stable.gpg
```

```bash
# Set some variables for repository setup
systemReleaseVersion=$(lsb_release -cs)
osLabel=$(lsb_release -sa 2>/dev/null|head -n 1|tr '[:upper:]' '[:lower:]')
```

```bash
# Set some variables for repository setup
systemReleaseVersion=$(lsb_release -cs)
osLabel=$(lsb_release -sa 2>/dev/null|head -n 1|tr '[:upper:]' '[:lower:]')
```

Adding the Source Repositories 
Once the basics have been installed we can set up the PowerDNS Repositories.

```bash
# PowerDNS Authoritative Repository
repo_pdnsAuth="deb [arch=amd64] http://repo.powerdns.com/${osLabel} ${systemReleaseVersion}-auth-48 main"
echo $repo_pdnsAuth | sudo tee "/etc/apt/sources.list.d/pdns-auth.list"

# PowerDNS Recursor Repository
repo_pdnsRec="deb [arch=amd64] http://repo.powerdns.com/${osLabel} ${systemReleaseVersion}-rec-48 main"
echo $repo_pdnsRec | sudo tee "/etc/apt/sources.list.d/pdns-rec.list"

# PowerDNS DNS Dist Repository
repo_dnsdist="deb [arch=amd64] http://repo.powerdns.com/${osLabel} ${systemReleaseVersion}-dnsdist-18 main"
echo $repo_dnsdist | sudo tee "/etc/apt/sources.list.d/dnsdist.list"
```

```bash
cat << EOF | sudo tee /etc/apt/preferences.d/pdns
Package: pdns-*
Pin: origin repo.powerdns.com
Pin-Priority: 600
EOF
```
