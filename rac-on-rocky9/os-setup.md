# OS Configuration for Oracle 19c RAC Nodes (Rocky Linux 9)

This document outlines the OS-level configuration required for deploying Oracle Grid Infrastructure and RAC on Rocky Linux 9.4/9.5 virtual machines.

---

## 1. Basic Shell and Editor Settings

Apply basic editor configuration and shell aliases for ease of use.

### For all users:

```bash
echo set tabstop=4 > ~/.vimrc
echo set shiftwidth=4 >> ~/.vimrc

su -
echo set tabstop=4 > ~/.vimrc
echo set shiftwidth=4 >> ~/.vimrc
```

### Disable SELinux and Configure Grub
```
grubby --update-kernel ALL --args selinux=0

grub2-setpassword  # Set GRUB boot password

# Disable Transparent HugePages
sed -i 's/quiet"/quiet transparent_hugepage=never"/' /etc/default/grub
grub2-mkconfig --update-bls-cmdline -o /boot/grub2/grub.cfg
```

### sudoers and Aliases
```
visudo  # Add sudo permissions
oracle  ALL=(ALL)   ALL
grid    ALL=(ALL)   ALL
```

### ulimit and Environment for oracle/grid
```
Append to /etc/profile:

if [ "$USER" = "oracle" ] || [ "$USER" = "grid" ]; then
  if [ "$SHELL" = "/bin/ksh" ]; then
    ulimit -p 16384
    ulimit -n 65536
  else
    ulimit -u 16384 -n 65536
  fi
  umask 022
fi
```

### Disable Firewalld and Avahi
```
systemctl stop firewalld
systemctl disable firewalld
systemctl stop avahi-daemon
systemctl disable avahi-daemon
```

### Base Package Installation
```
dnf makecache

dnf install -y \
  bc binutils elfutils-libelf elfutils-libelf-devel fontconfig-devel \
  glibc glibc-devel ksh libaio libaio-devel libXrender libX11 libXau libXi \
  libXtst libgcc libnsl librdmacm libstdc++ libstdc++-devel libxcb \
  libibverbs make smartmontools sysstat gcc gcc-c++ policycoreutils \
  policycoreutils-python-utils nfs-utils openssh-server mlocate samba \
  sysstat chkconfig

dnf clean all
```

### Transparent HugePages / No Zeroconf
```
echo NOZEROCONF=yes >> /etc/sysconfig/network
cat /sys/kernel/mm/transparent_hugepage/enabled
```

### Samba Share Setup (Optional)
```
mkdir -p -m 777 /home/public

cp /etc/samba/smb.conf /etc/samba/smb.conf.org

# Edit /etc/samba/smb.conf
[public]
    path = /home/public
    writable = yes
    guest ok = yes
    guest only = yes
    create mode = 0777
    directory mode = 0777

systemctl enable --now smb nmb
```

### Kernel Parameter Tuning (HugePages, shmmax, etc.)
```
MEMTOTAL=$(free -b | sed -n '2p' | awk '{print $2}')
SHMMAX=$(expr $MEMTOTAL / 2)
SHMMNI=4096
PAGESIZE=$(getconf PAGE_SIZE)

cat > /etc/sysctl.conf << EOF
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmmax = $SHMMAX
kernel.shmall = $(expr $MEMTOTAL / $PAGESIZE)
kernel.shmmni = $SHMMNI
kernel.sem = 250 32000 100 128
kernel.panic_on_oops = 1
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
vm.nr_hugepages = 2370
EOF

/sbin/sysctl --system
```

### User and Group Creation
```
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
groupadd -g 54324 backupdba
groupadd -g 54325 dgdba
groupadd -g 54326 kmdba
groupadd -g 54327 asmdba
groupadd -g 54328 asmoper
groupadd -g 54329 asmadmin
groupadd -g 54330 racdba

useradd -u 1001 -g oinstall -G asmadmin,asmdba,asmoper,racdba grid
useradd -u 1002 -g oinstall -G dba,oper,backupdba,dgdba,kmdba,asmdba,racdba oracle

passwd oracle
passwd grid

mkdir -p /u01/app/grid
mkdir -p /u01/app/19.3.0/grid
mkdir -p /u01/app/oracle/product/19.3.0/dbhome_1
mkdir -p /u01/app/oraInventory

chown -R grid:oinstall /u01
chown -R oracle:oinstall /u01/app/oracle
chown -R grid:oinstall /u01/app/oraInventory
chmod -R 775 /u01
```

### User Limits
```
Edit /etc/security/limits.conf and /etc/pam.d/login:

# For grid and oracle
<user> soft nproc 2047
<user> hard nproc 16384
<user> soft nofile 1024
<user> hard nofile 65536
<user> soft stack 10240
<user> hard stack 32768
<user> soft memlock unlimited
<user> hard memlock unlimited

echo "session required pam_limits.so" >> /etc/pam.d/login
```

### Disable chronyd / Adjust Hosts File
```
systemctl stop chronyd
systemctl disable chronyd
mv /etc/chrony.conf /etc/chrony.conf.org

Example /etc/hosts entry:

# node01
10.0.10.101 node01.ha.jp node01
10.0.10.111 node01-vip.ha.jp node01-vip
10.0.20.101 node01-priv.ha.jp node01-priv
# node02
10.0.10.102 node02.ha.jp node02
10.0.10.112 node02-vip.ha.jp node02-vip
10.0.20.102 node02-priv.ha.jp node02-priv
```

### Disk Permissions via Udev Rules
```
vi /etc/udev/rules.d/99-oracle.rules
KERNEL=="sd[b-z]",ACTION=="add|change",OWNER="grid",GROUP="asmadmin",MODE="0660"

/sbin/udevadm trigger --type=devices --action=add
```

### VM Cloning and Hostname Reset
```
hostnamectl set-hostname node02.ha.jp
reboot
```
