<h1>password-hash-cracking</h1>
<h2>Description</h2>
password hash identification and cracking analysis using john the ripper, with defensive recommendation for hash storage and detection.
<br/>

<h2>Overview</h2>
this lab focuses on understanding how attackers recover plaintext passwords from captured hash values using john the ripper, one of the most widely used offline password-cracking tools. the goal was to practice hash identification, apply different cracking strategies (dictionary, single-crack mode, rule-based mangling), and translate offensive findings into defensive recommendations relevant to a soc context.

<h2>Tools Used</h2>

- <b>John the Ripper</b>

- <b>rockyou.txt wordlist from the massive 2009 data breach of "RockYou"<b/>

- <b>zip2john / rar2john<b/>

- <b>Linux CLI (Kali)<b/>

<h2>Methodology</h2>
1. Hash Identification
<br/>
<br/>
before cracking, each hash was analyzed to determine its type (e.g., MD5, SHA-1, sha512crypt) based on format, length, and structure. Correct identification is critical, supplying the wrong '--format' flag to John will cause cracking attempts to fail silently or produce false negatives.

2\. Dictionary Attacks<br/>

using the rockyou.txt wordlist, i ran standard dictionary attacks against sample hashes: <br/>

&emsp;john --format=<hash-type> --wordlist=rockyou.txt hash.txt

this method is effective against weak or commonly reused passwords and demonstrated why breach-derived wordlists remain dangerous years later.

3\. Single Crack Mode <br/>

single crack mode uses contextual information (e.g., username) to generate target password guesses, useful when a password is likely derived from account metadata:

&emsp;john --single --format=<hash-type> hash.txt

4\. Cracking Protected Archives <br/>

used zip2john and rar2john to extract crackable hash representations from password protected zip/rar files, then applied john to recover the archive password demonstrating that "encryption" on consumer archive tools is often only as strong as the password behind it.

5\. Cracking Windows Authentication Hashes <br/>

windows stores password hashes in NTLM format, which (unlike modern Linux sha512crypt hashes) is unsalted, making it significantly faster to crack at scale. after identifying the hash format, i applied john using the appropriate NTLM setting against a wordlist:

&emsp;john --format=NT --wordlist=rockyou.txt hash.txt

this highlights a key legacy weakness in NTLM: because there's no salt, identical password across different accounts produces identical hash values, meaning, a single successful crack or a precomputed hash table can compromise multiple accounts at once.

6\. Cracking SSH Private Key Passwords <br/>

SSH private keys are often protected with a passphrase rather than stored in plaintext. using ss2john, i converted a passphrase-protected private key into a crackable hash format, then ran john against it:

&emsp;ssd2john id_rsa > id_rsa_hash.txt<br/>
&emsp;john --wordlist=rockyou.txt id_rsa_hash.txt

this demonstrates that even non-traditional "password" scenarios are subject to the same offline cracking risks if the passphrase is weak.

<h2>Findings</h2>

- several hashes were cracked within seconds using a standard wordlist, reinforcing how quickly weak or reused passwords fall to common tools.

- hash type misidentification was the most common source of failed cracking attempts because correct format selection matters as much as wordlist quality.

- archive and ssh key passwords, while often overlooked, are just as vulnerable to offline cracking as traditional account hashes if the underlying password is weak.

- NTLM's lack of salting made windows-style hashes crack noticeable faster than salted formats, reinforcing why legacy authentication schemes remain a liability even in modern environments.

<h2>Defensive Takeaways</h2>

- hashing algorithm matters: md5/sha-1 are fast to crack at scale. modern systems should use slow, salted algorithms (bcrypt, scrypt, etc.) to resist offline attacks.

- password policy: length and complexity requirements reduce dictionary attack success far more than periodic rotation policies do.

- detection opportunity: while this lab focused on offline cracking (generates no network or log signal), the same credential weaknesses are what enable online attacks like credential stuffing and password spraying (e.g., windows event id 4625, ssh auth failures).

- environments still relying on NTLM authentication (instead of kerberos) remain exposed to fast offline cracking and hash relay attachs; disabling NTLM where possible, or at minimum monitoring for NTLM authentication events, reduces this exposure.

- recommendations: a siem correlation rule alerting on n failed logins across multiple accounts from a single source in a short window would help catch spraying attempts stemming from a breached password list.

<h2>Dictionary Attack Walkthrough</h2>
Scope the files/hashes to be cracked:<br/>
<img src="https://imgur.com/yyMUKfC.png" height="80%" width="80%" alt="Hash Identification"/> <br/>
<img src="https://imgur.com/m77UzHB.png" height="80%" width="80%" alt="Hash Identification"/> <br/>

- run the "cat" or concatenate command to print the contents of hash1.txt
<br/>

Identify the hash using "HashID" by Blackploit: <br/>
<img src="https://imgur.com/RrA7DHd.png" height="80%" width="80%" alt="Hash Identification"/> <br/>

- run the python code using the command python hash-id.py and input the hash from hash1.txt
- the output tells us that the possible hash could be "MD5".
<br/>

Find the correct MD5 format to use for John the Ripper: <br/>
<img src="https://imgur.com/CwYZiBE.png" height="80%" width="80%" alt="Hash Identification"/> <br/>

- the output of cat hash1.txt showed us that the hash is a standard, unformatted, and unsalted MD5 hash. thus, it is raw.
- we will use "Raw-MD5".
<br/>

Crack the Hash: <br/>
<img src="https://imgur.com/COudWEW.png" height="80%" width="80%" alt="Hash Identification"/> <br/>

- run the command: john --format=[format] --wordlist=[path to wordlist] [path to file]
- the cracked hash is displayed within the yellow box [redacted].

<h2>Single Crack Mode Walkthrough</h2>
Scope the files/hashes to be cracked:<br/>
<img src="https://imgur.com/L1kN3FV.png" height="80%" width="80%" alt="Hash Identification"/> <br/>
<img src="https://imgur.com/Q9z8clZ.png" height="80%" width="80%" alt="Hash Identification"/> <br/>

- we will be cracking the password of user "Joker".
- run the "cat" or concatenate command to print the contents of hash07.txt
<br/>

Identify the hash using "HashID" by Blackploit: <br/>
<img src="https://imgur.com/obW8sTB.png" height="80%" width="80%" alt="Hash Identification"/> <br/>

- run the python code using the command python hash-id.py and input the hash from hash07.txt
- the output tells us the hash is Raw-MD5.
<br/>

Prepend the username "Joker" before the password hash in hash07.txt: <br/>
<img src="https://imgur.com/ZvBrCJB.png" height="80%" width="80%" alt="Hash Identification"/> <br/>

- use the command "nano hash07.txt" to open the .txt file with the command line text editor in linux.
<br/>

Crack the Hash with the username: <br/>
<img src="https://imgur.com/XgOIHkT.png" height="80%" width="80%" alt="Hash Identification"/> <br/>

- run the command: john --single=[format] [path to file]
- the cracked hash is displayed within the yellow box.
<br/>




