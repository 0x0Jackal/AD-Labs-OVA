## Support AD-Lab

## SMB Enumeration 
The SMB service was discovered on the target machine. To identify accessible shares, anonymous authentication was attempted.

smbclient -L //IP/ -N

The enumeration revealed a share named Config2. The share was accessed using the following command:

smbclient //IP/Config2 -N

Inside the share, a file named: creds.txt. This file contained credentials for a low-privileged domain user, which can be used for further enumeration.


## LDAP Enumeration
Using the discovered credentials, LDAP enumeration allows retrieve domain user information.

ldapsearch -x -H ldap://IP -D "DOMAIN\\lowuser" -w 'password' -b "DC=domain,DC=local" "(objectClass=user)"

A user named "support" was identified. Inspecting the attributes of this account revealed that the info attribute contained the plaintext password for the support user.


## AS-REP Roasting
Using the credentials of the support account, AS-REP roasting was performed to identify users configured without Kerberos pre-authentication.

impacket-GetNPUsers DOMAIN.LOCAL/support:'password' -dc-ip IP -request

This command requests AS-REP responses for accounts that have the "Do not require Kerberos preauthentication" option enabled.
The attack successfully returned an AS-REP hash for the Administrator account.

## Cracking the AS-REP Hash
The obtained hash can be cracked offline using Hashcat.

hashcat -m 18200 asrep_hash.txt /usr/share/wordlists/rockyou.txt (Use the provided wordlist in this repo)
