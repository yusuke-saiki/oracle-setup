# Oracle Grid Infrastructure Installation (19.3.0 + RU) on Rocky Linux 9

This section describes the installation of Oracle Grid Infrastructure 19c (version 19.3.0) with a patch (Release Update) applied, as part of Oracle RAC setup on Rocky Linux 9.

---

## 1. Extract the Grid Infrastructure ZIP

As `grid` user on **node01**:

```bash
cd $ORACLE_HOME
mv /home/public/V982068-01.zip .
unzip V982068-01.zip
rm -f V982068-01.zip
```

## 2. Update OPatch Utility
```
mv OPatch OPatch_old
mkdir patch
cd patch
cp /home/public/p6880880_190000_Linux-x86-64.zip .
unzip p6880880_190000_Linux-x86-64.zip
rm -f p6880880_190000_Linux-x86-64.zip
mv OPatch ../
```

## 3. Prepare GI Release Update Patch (RU)
```
cd /var/tmp
mv /home/public/p37257886_190000_Linux-x86-64.zip .
unzip p37257886_190000_Linux-x86-64.zip
rm -f p37257886_190000_Linux-x86-64.zip
```

## 4. Enable SSH Equivalence Between Nodes
```
On both nodes (node01 & node02) as grid:
ssh-keygen -t rsa

On node02:
ssh-copy-id grid@node01.ha.jp

On node01:
ssh-copy-id grid@node02.ha.jp
```

## 5. Fix INS-06006 (Optional Workaround)
```
On node01, as root:

cp -p /usr/bin/scp /usr/bin/scp.bkp
echo "/usr/bin/scp.bkp -T \$*" > /usr/bin/scp
chmod +x /usr/bin/scp
```

## 6. CVU Disk RPM Setup
```
On node01:

mkdir /root/cv
cp -p /u01/app/19.3.0/grid/cv/rpm/* /root/cv/
scp /root/cv/* root@node02:/root/cv/
chown grid:oinstall /root/cv/*
CVUQDISK_GRP=oinstall; export CVUQDISK_GRP
rpm -ivh /root/cv/*

On node02:

rpm -ivh /root/cv/*
```

## 7. compat-* Package Setup (For Compatibility)
```
On node01:

mkdir /root/compat
mv /home/public/compat* /root/compat/
scp /root/compat/* root@node02:/root/compat/
rpm -ivh /root/compat/*libcap*
rpm -ivh /root/compat/*libstdc*

Repeat the same rpm -ivh commands on node02.
```

## 8. Cluster Validation
```
export CV_ASSUME_DISTID=OL7
$ORACLE_HOME/runcluvfy.sh stage -pre crsinst -n node01,node02 -verbose
```

## 9. Launch Grid Infrastructure Installer
```
As grid on node01:

cd $ORACLE_HOME
export LANG=C
export CV_ASSUME_DISTID=OL7
./gridSetup.sh -applyRU /var/tmp/37257886/
```

## 10. Verify Cluster Status
```
crsctl stat res -t
```
