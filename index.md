<p align='center'>
  <img src='assets/img/logo.png' width='350px' />
</p>
<!-- ![deadface logo](./logo_deadface_2021.pdddng) -->

## Badges
<div align='center'>
  <img src='assets/img/competitor.png' width='150px' />
  <img src='assets/img/author.png' width='150px' />
</div>

## Challenges 
---
### Forensics
#### Windows 1
##### Solve
Download and extracting the memory dump then inspecting the image with volatility showed the operating system, version and date of the image.

```bash
$ volatility -f ./physmemraw imageinfo
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win10x64_19041
                     AS Layer1 : SkipDuplicatesAMD64PagedMemory (Kernel AS)
                     AS Layer2 : FileAddressSpace (/dumps/physmemraw)
                      PAE type : No PAE
                           DTB : 0x1aa000L
                          KDBG : 0xf8005e600b20L
          Number of Processors : 4
     Image Type (Service Pack) : 0
                KPCR for CPU 0 : 0xfffff8005ba60000L
                KPCR for CPU 1 : 0xffff82804f9c0000L
                KPCR for CPU 2 : 0xffff82804f5e8000L
                KPCR for CPU 3 : 0xffff82804f7ca000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2021-09-07 14:57:44 UTC+0000
     Image local date and time : 2021-09-07 07:57:44 -0700
```
Taking this infomration we get the flag 
##### Flag
`flag{Win10_x64_202109071457}`

---

#### Bloody Bash 1 & 2
##### Solve
My solve for this probably wasnt the intended method, but it worked
```bash 
grep -iR flag /home
```
This provided back the flags for `bloody bash 1` and `bloody bash 2`
##### Flag
`flag{cd134eb8fbd794d4065dcd7cfa7efa6f3ff111fe}`
`flag{a856b162978fe563537c6890cb184c48fc2a018a}`

---

#### Bloody Bash 3
##### Solve
This was solved by finding the root elevation method by listing out users sudo rules
```bash
$ sudo -l 
Matching Defaults entries for bl0ody_mary on ad3c9aa87fe2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bl0ody_mary may run the following commands on ad3c9aa87fe2:
    (ALL) NOPASSWD: /opt/start.sh, /usr/sbin/srv
```
From this list there is 2 paths that are interesting. Checking out the first `/opt/start.sh` 
```bash
$ cat /opt/start.sh
#!/bin/bash

sudo /usr/sbin/srv &
exec /bin/bash
```
The script has an `execve` for a shell instance which give you a root shell. 

Using the root access method from `bloody bash 3` and looking at the script at `/opt/start.sh` we can see that there is a call made, and pushed to background, that starts up a listener script. Looking at the script you can see that there is a kex string. Decoding the string provides the final flag for this container.
```python
flag = '666c61677b6f70656e5f706f727428616c29737d'
print( bytes.fromhex(flag).decode() )"
```
##### Flag
`flag{open_port(al)s}`

---

#### Bloody Bash 4
##### Solve 
Looking for the flag in the remainder of the container didnt return any results. Looking back at user `bl0ody_mary` home directory there is a PDF file. `strings` doesnt return anything (installed via apt as no restrictions to network) and `grep` has the same result on the file for `flag`. 

There is no way to view the PDF locally as its a remote terminal so I tried to copy the file back with `scp` but was not able to as the user. This was probably a permission issue in sshd_config but I didnt want to look at that as it was a single file that wasnt too big. 

Base64 encoding the PDF and copying via clipboard to my local machine allowed me to get the PDF local and view in a PDF reader.

```bash
$ cat De\ Monne\ Customer\ Portal.pdf | base64 -w0
```
Taking this data to my local machine to rebuild the raw PDF
```bash
$ cat <EOF >x.pdf 
...
...
...data.
...
...
EOF

$ xdg-open x.pdf 
```
The flag should be visible in the PDF document.

##### Flag 
`flag{deM0nn3_dat4_4_us}`

---
---

### Exploitation
#### Old Devil
##### Solve 
A single binary that asks for the devils name. If its provided a long enough input it will overflow and dump the flag. 
```bash
$ ./deamon

Luciafer v1.0
Say the demon's name to gain access to the secret.
Enter the demon's name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

That is not the demon's name.
flag{AdraMMel3ch}
zsh: segmentation fault  ./demon
```
##### Flag
`flag{AdraMMel3ch}`

---

#### Password Insecurities
##### Description
Players are presented with a mysql databased dump and asked to get the flag for a particular user.
##### Solve
Getting the users hash from the database is a simple 
```sql
select cust_passwd from customer where first_name = x and last_name = y;
```
The hash is a md5crypt `$1$salt$password` format. Dump the hash into a file and use john with rockyou to get the password
```bash
$ echo '$1$FigUPHDJ$IYWZKYxoKDdLyODRM.kQq.' > hash
$ john hash /usr/share/wordlists/rockyou.txt
$ john hash /usr/share/wordlists/rockyou.txt --show
?:trustno1
```
##### Flag
`flag{trustno1}`
