# Blazor AD-Labs-OVA Writeup (Real Life Scenario)

1-) Nmap scan to identify open services on the target machine.

nmap -sC -sV -p-

<img width="1566" height="867" alt="2026-03-13 21_40_41-KALI  Running  - Oracle VirtualBox" src="https://github.com/user-attachments/assets/b2b3f256-0e63-455e-ac99-68033cda86c1" />

2-) Accessing the service in the browser: http://ip:8080
The application appeared to be running Blazor WebAssembly.
Blazor WebAssembly applications load their logic in .wasm and .dll files, which are downloaded by the client browser.
Several files were loaded dynamically by the application. One file in particular stood out:
BlazorCredintialLeak.wasm

<img width="1881" height="834" alt="Screenshot 2026-03-13 214147" src="https://github.com/user-attachments/assets/fd593272-b1f6-4d80-89b8-fa819a38046c" />

3-) After downloading the file, it was analyzed locally.
Inside the WebAssembly file, hardcoded credentials were discovered:
username: blazor
password: blazor123@

<img width="1920" height="868" alt="2026-03-13 21_42_20-BlazorCredentialLeak(1) wasm - Notepad" src="https://github.com/user-attachments/assets/3759a86b-6f02-454e-984c-8ccaf60094ff" />

4-) The discovered credentials were tested against the SMB service using NetExec (nxc).

<img width="1559" height="164" alt="2026-03-13 21_43_09-KALI  Running  - Oracle VirtualBox" src="https://github.com/user-attachments/assets/94f8a1ec-f21c-4b99-815d-7b8e8b9eacef" />

The authentication was successful, confirming that the credentials were valid for the domain environment.
This provided authenticated access to the Active Directory network.

5-) Kerberoasting Attack

After obtaining valid domain credentials, the next step was to enumerate Service Principal Names (SPNs).
SPN accounts are often associated with services and can sometimes contain weak passwords, making them good candidates for Kerberoasting attacks.
The following command was used: This command requested Kerberos service tickets for SPN-enabled accounts and returned a TGS hash.

impacket-GetUserSPNs domain.local/blazor:'blazor123@' -dc-ip <TARGET_IP> -request

<img width="1863" height="525" alt="2026-03-13 19_34_36-KALI  Running  - Oracle VirtualBox" src="https://github.com/user-attachments/assets/3b0b99ac-3abb-43ea-84b0-642c5dcc0c49" />

6-) Password Cracking
The extracted Kerberos ticket hash was saved to a file and cracked using Hashcat.

hashcat -m 13100 spn_hash.txt /usr/share/wordlists/rockyou.txt [Please use the attached wordlist for craking in the repo]

<img width="1866" height="724" alt="2026-03-13 19_35_27-KALI  Running  - Oracle VirtualBox" src="https://github.com/user-attachments/assets/7a3981e2-e873-43c2-b005-0ff0cf71d66c" />
