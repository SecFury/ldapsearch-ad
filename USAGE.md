# ldapsearch-ad.py detailled usage

## -h could help

``` text
$ ./ldapsearch-ad.py -h
usage: ldapsearch-ad.py [-h] [-l LDAP_SERVER] [-ssl] [-t REQUEST_TYPE] [-d DOMAIN] [-u USERNAME] [-p PASSWORD]
                        [-H HASHES] [-s SEARCH_FILTER] [-z SIZE_LIMIT] [-o OUTPUT_FILE] [-v] [--version]
                        [search_attributes [search_attributes ...]]

Active Directory LDAP Enumerator

positional arguments:
  search_attributes     LDAP attributes to look for (default is all).

optional arguments:
  -h, --help            show this help message and exit
  -l LDAP_SERVER, --server LDAP_SERVER
                        IP address of the LDAP server.
  -ssl, --ssl           Force an SSL connection?.
  -t REQUEST_TYPE, --type REQUEST_TYPE
                        Request type: info, whoami, search, trusts, pass-pols, admins, show-user, show-user-list,
                        kerberoast, search-spn, asreproast, goldenticket, search-delegation, createsid, all
  -d DOMAIN, --domain DOMAIN
                        Authentication account's FQDN. Example: "contoso.local".
  -u USERNAME, --username USERNAME
                        Authentication account's username.
  -p PASSWORD, --password PASSWORD
                        Authentication account's password.
  -H HASHES, -hashes HASHES
                        NTLM hashes, format is LMHASH:NTHASH
  -s SEARCH_FILTER, --search-filter SEARCH_FILTER
                        Search filter (use LDAP format).
  -z SIZE_LIMIT, --size_limit SIZE_LIMIT
                        Size limit (default is 100, or server' own limit).
  -o OUTPUT_FILE, --output OUTPUT_FILE
                        Write results in specified file too.
  -v, --verbose         Turn on debug mode
  --version             Show version and exit
```

## Actions

### -t info

Retrieve server **information** without credentials using `-t info`.
Usefull to get the FQDN and the functionality levels.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.20 -t info
### Server infos ###
[+] Forest functionality level = Windows 2012 R2
[+] Domain functionality level = Windows 2012 R2
[+] Domain controller functionality level = Windows 2012 R2
[+] rootDomainNamingContext = DC=evilcorp,DC=lab2
[+] defaultNamingContext = DC=evilcorp,DC=lab2
[+] ldapServiceName = evilcorp.lab2:mtldc1$@EVILCORP.LAB2
[+] naming_contexts = ['DC=evilcorp,DC=lab2', 'CN=Configuration,DC=evilcorp,DC=lab2', 'CN=Schema,CN=Configuration,DC=evilcorp,DC=lab2', 'DC=DomainDnsZones,DC=evilcorp,DC=lab2', 'DC=ForestDnsZones,DC=evilcorp,DC=lab2']
```

### -t whoami

Check authentication using `-t whoami`.
Usefull to attempt an authentication to verify a password.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u bbilly -p 'P@$$w0rd' -t whoami
### Result of "whoami" command ###
[+] u:EVILCORP\bbilly
```

### -t trusts

List **trusts** attributes using `-t trusts` (user account needed).
Usefull to get a full view of the Active Directory overall architecture.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u jjohnny -p 'P@$$word' -t trusts
### Result of "trusts" command ###
[+] + fra.evilcorp.lab2 (FRA)
[+] |___trustAttributes = ['TRUST_ATTRIBUTE_WITHIN_FOREST']
[+] |___trustDirection = Bidirectional
[+] |___trustType = The trusted domain is a Windows domain running Active Directory.
[+] |___trustPartner = fra.evilcorp.lab2
[+] |___securityIdentifier = S-1-5-21-2894840767-735700-3593130334
[+] |___whenCreated = 2019-03-09 04:57:15+00:00
[+] |___whenChanged = 2019-03-09 04:57:15+00:00
[+] + total.lab2 (TOTAL)
[+] |___trustAttributes = []
[+] |___trustDirection = Outbound
[+] |___trustType = The trusted domain is a Windows domain running Active Directory.
[+] |___trustPartner = total.lab2
[+] |___securityIdentifier = S-1-5-21-2894840767-735700-3503349313
[+] |___whenCreated = 2018-11-05 11:51:18+00:00
[+] |___whenChanged = 2022-02-11 20:23:40+00:00
```

### -t pass-pols

List **password policies** using `-t pass-pols` (user account needed for default password policy / admin account needed for fine grained password policies).
Usefull to prepare our next password spraying attack ;)

``` text
$ ./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u jjohnny -p 'P@$$word' -t pass-pols
### Result of "pass-pols" command ###
[+] Default password policy:
[+] |___Minimum password length = 10
[+] |___Password complexity = Enabled
[*] |___Lockout threshold = 12
[+] |___  Lockout duration = 30 minutes, 0 seconds
[+] |___  Lockout observation window = 30 minutes, 0 seconds
[*] |___Password history length = 8
[+] |___Max password age = 120 days, 0 hours, 0 minutes, 0 seconds
[+] |___Min password age = 0 seconds
[+] No fine grained password policy found (high privileges are required).
```

### -t admins

Show the **admins** (members of *domain admins*, *enterprise admins*, *administrators*, and their french equivalent) and their most interesting flags using `-t admins` (user account needed).
Usefull to find juicy targets.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.2 -d evilcorp.lab2 -u jjohnny -p 'P@$$word' -t admins     
### Result of "admins" command ###
[+] All members of group "Administrateurs":
[*]     admbilly (DONT_EXPIRE_PASSWORD)
[+]     Administrateur
[+] All members of group "Admins du domaine":
[+]     Administrateur
[+] All members of group "Administrateurs de l’entreprise":
[*]     admbilly (DONT_EXPIRE_PASSWORD)
```

### -t kerberoast

Show the user accounts vunerable to **kerberoast** attacks.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.2 -d evilcorp.lab2 -u jjohnny -p 'P@$$word' -t kerberoast     
### Result of "kerberoast" command ###
[*] admDupond : MSSQLSvc/srv-db-01.evilcorp.lab2:18739, MSSQLSvc/srv-db-01.evilcorp.lab2:65239
[*] admDurand : HTTP/192.168.56.212, HTTP/srv-web-01.evilcorp.lab2, HTTP/srv-web-01
```

References:

* <https://hackndo.com/kerberoasting/>

### -t asreproast

Show the user accounts vunerable to **asreproast** attacks.

I cannot create the proper documentation for the moment.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.2 -d evilcorp.lab2 -u jjohnny -p 'P@$$word' -t asreproast     
### Result of "asreproast" command ###
TO BE COMPLETED
```

### -t search-spn

Search of account having specific XXX.
It is possible to search for services starting with the `-s` parameter.

Example to look for HTTP services:

``` text
$ ./ldapsearch-ad.py -l 192.168.56.2 -d evilcorp.lab2 -u jjohnny -p 'P@$$word' -t search-spn -s 'http'
### Result of "search-spn" command ###
[*] admDupond : HTTP/192.168.56.221, HTTP/srv-web-01.evilcorp.lab2, HTTP/srv-web-01
[*] admDeschamps : HTTP/192.168.56.222, HTTP/srv-web-02.evilcorp.lab2, HTTP/srv-web-02
```

### -t goldenticket

`-t goldenticket` retrieves the last time the krbtgt password changed.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.2 -d evilcorp.lab2 -u jjohnny -p 'P@$$word' -t goldenticket
### Result of "goldenticket" command ###
[+] krbtgt password changed at 2021-02-26 19:25:41
```

### -t search-delegation

`-t search-delegation` list all the account which are trusted for delegation (*TRUSTED_FOR_DELEGATION* flag set).

``` text
$ ./ldapsearch-ad.py -l 192.168.56.2 -d evilcorp.lab2 -u jjohnny -p 'P@$$word' -t search-delegation
### Result of "search-delegation" command ###
[*] srv-ad-01$
[*] srv-ad-02$
[*] admbilly
```

References:

* <https://posts.specterops.io/hunting-in-active-directory-unconstrained-delegation-forests-trusts-71f2b33688e1?gi=52b572a3db09>
* <https://cheatsheet.haax.fr/windows-systems/privilege-escalation/delegations/>

### -t createsid

Get info about createsid from ms-ds-creatorsid.

I need to fully understand what it looks for before making this documentation.

### -t all

Retrieve all interesting information with a simple user account using `-t all`.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.2 -d evilcorp.lab2 -u jjohnny -p 'P@$$word' -t all 
### Server infos ###
[+] Forest functionality level = Windows 2012 R2
[+] Domain functionality level = Windows 2012 R2
[+] Domain controller functionality level = Windows 2012 R2
[+] rootDomainNamingContext = DC=evilcorp,DC=lab2
[+] defaultNamingContext = DC=evilcorp,DC=lab2
[+] ldapServiceName = evilcorp.lab2:win2012-dc01$@EVILCORP.LAB2
[+] naming_contexts = ['DC=evilcorp,DC=lab2', 'CN=Configuration,DC=evilcorp,DC=lab2', 'CN=Schema,CN=Configuration,DC=evilcorp,DC=lab2', 'DC=DomainDnsZones,DC=evilcorp,DC=lab2', 'DC=ForestDnsZones,DC=evilcorp,DC=lab2']
### Result of "trusts" command ###
[+] + fra.evilcorp.lab2 (FRA)
[+] |___trustAttributes = ['TRUST_ATTRIBUTE_WITHIN_FOREST']
[+] |___trustDirection = Bidirectional
[+] |___trustType = The trusted domain is a Windows domain running Active Directory.
[+] |___trustPartner = fra.evilcorp.lab2
[+] |___securityIdentifier = S-1-5-21-2894840767-735700-3593130334
[+] |___whenCreated = 2019-03-09 04:57:15+00:00
[+] |___whenChanged = 2019-03-09 04:57:15+00:00
[+] + total.lab2 (TOTAL)
[+] |___trustAttributes = []
[+] |___trustDirection = Outbound
[+] |___trustType = The trusted domain is a Windows domain running Active Directory.
[+] |___trustPartner = total.lab2
[+] |___securityIdentifier = S-1-5-21-2894840767-735700-3503349313
[+] |___whenCreated = 2018-11-05 11:51:18+00:00
[+] |___whenChanged = 2022-02-11 20:23:40+00:00
### Result of "pass-pols" command ###
[+] Default password policy:
[+] |___Minimum password length = 10
[+] |___Password complexity = Enabled
[*] |___Lockout threshold = 12
[+] |___  Lockout duration = 30 minutes, 0 seconds
[+] |___  Lockout observation window = 30 minutes, 0 seconds
[*] |___Password history length = 8
[+] |___Max password age = 120 days, 0 hours, 0 minutes, 0 seconds
[+] |___Min password age = 0 seconds
[+] No fine grained password policy found (high privileges are required).
### Result of "admins" command ###
[+] All members of group "Administrateurs":
[*]     admbilly (DONT_EXPIRE_PASSWORD)
[+]     Administrateur
[+] All members of group "Admins du domaine":
[+]     Administrateur
[+] All members of group "Administrateurs de l’entreprise":
[*]     admbilly (DONT_EXPIRE_PASSWORD)
### Result of "kerberoast" command ###
[*] admDupond : MSSQLSvc/srv-db-01.evilcorp.lab2:18739, MSSQLSvc/srv-db-01.evilcorp.lab2:65239
[*] admDurand : HTTP/192.168.56.212, HTTP/srv-web-01.evilcorp.lab2, HTTP/srv-web-01
### Result of "asreqroast" command ###
### Result of "goldenticket" command ###
[+] krbtgt password changed at 2021-02-26 19:25:41
```

### -t show-user

Show the most interesting attributes of a user using `-t show-user` (user account needed).
Usefull to get more information about a juicy target.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u jjohnny -p 'P@$$word' -t show-user -s '(samaccountname=adm*)'
### Result of "show-user" command ###
[+] admbilly
[+] |___type: user
[+] |___description = ['First admin']
[*] |___The adminCount is set to 1
[+] |___userAccountControl = ACCOUNTDISABLE, NORMAL_ACCOUNT
[+] |___sAMAccountType = SAM_USER_OBJECT
[+] |___memberOf = grp-gods, grp-passwords-never-expire
[+] admwilly
[+] |___type: user
[+] |___displayName = admwilly
[+] |___description = ['Second admin']
[*] |___The adminCount is set to 1
[+] |___userAccountControl = NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
[+] |___sAMAccountType = SAM_USER_OBJECT
[+] |___memberOf = grp-gods, grp-vmware-admins
```

or even computers or groups. Everything depend of the search parameter `-s`.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u jjohnny -p 'P@$$word' -t show-user -s '(samaccountname=srv-ad*)'
### Result of "show-user" command ###
[+] srv-ad-01$
[+] |___type: computer
[+] |___description = ['First DC']
[+] |___userAccountControl = SERVER_TRUST_ACCOUNT, TRUSTED_FOR_DELEGATION
[+] |___sAMAccountType = SAM_MACHINE_ACCOUNT
[+] srv-ad-02$
[+] |___type: computer
[+] |___displayName = SVP1-AD01273-1$
[+] |___description = ['First RODC']
[+] |___userAccountControl = WORKSTATION_TRUST_ACCOUNT, TRUSTED_TO_AUTH_FOR_DELEGATION, PARTIAL_SECRETS_ACCOUNT
[+] |___sAMAccountType = SAM_MACHINE_ACCOUNT
[+] |___memberOf = grp-rodc

$ ./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u jjohnny -p 'P@$$word' -t show-user -s '(cn=bad admins)'
### Result of "show-user" command ###
[+] bad_admins
[+] |___type: group
[+] |___description = ['Bad Admins']
[+] |___sAMAccountType = SAM_GROUP_OBJECT
[+] |___memberOf = Domain Admins
```

## Authenticate with an NTLM hash instead of a password

Because sometimes the compromise is still on-going.

``` bash
./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u jjohnny -hashes :32ed87bdb5fdc5e9cba88547376818d4 -t show-admins
```

## Append output to a file

It is possible to append the output in a file by using `-o <filename>`.
For instance:

``` bash
./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u jjohnny -p 'P@$$word' -t info -o info_from_DC.log
```

## Advanced usage using search

Search for any information using the powerful ldap filter syntax with `-t search`:
Only a few attributes are not shown because they seem useless, and a few other are interpreted.

It is possible to set the attributes to retrive at the end of the command line to reduce the search load.

``` text
$ ./ldapsearch-ad.py -l 192.168.56.20 -d evilcorp -u jjohnny -p 'P@$$word' -t search -s '(samaccountname=bbilly)' cn samaccountname useraccountcontrol memberof displayname samaccounttype
### Result of "search" command ###
[+] |___cn = bbilly
[+] |___displayName = BILLY Billy
[+] |___memberOf = ['CN=grp-gods,OU=Admins,DC=evilcorp,DC=lab2', 'CN=grp-another-example,OU=Groups,DC=evilcorp,DC=lab2']
[+] |___sAMAccountName = bbilly
[+] |___sAMAccountType = SAM_USER_OBJECT
[+] |___userAccountControl = NORMAL_ACCOUNT
```