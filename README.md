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
1. Hash Identification<br/><br/>

before cracking, each hash was analyzed to determine its type (e.g., MD5, SHA-1, sha512crypt) based on format, length, and structure. Correct identification is critical, supplying the wrong '--format' flag to John will cause cracking attempts to fail silently or produce false negatives.

2\. Dictionary Attacks<br/>

using the rockyou.txt wordlist, i ran standard dictionary attacks against sample hashes: <br/>

&emsp;john --format=<hash-type> --wordlist=rockyou.txt hash.txt

this method is effective against weak or commonly reused passwords and demonstrated why breach-derived wordlists remain dangerous years later.

3\. Single Crack Mode <br/>

single crack mode uses contextual information (e.g., username) to generate target password guesses, useful when a password is likely derived from account metadata:

&emsp;john --single --format=<hash-type> hash.txt

4\. Cracking Linux Shadow Hashes <br/>

practiced extracting and combining relevant fields from /etc/passwd and /etc/shadow into a crackable format, then applying john with the appropriate format (e.g., sha512crypt) to recover weak local account passwords.

5\. Cracking Protected Archives <br/>

used zip2john and rar2john to extract crackable hash representations from password protected zip/rar files, then applied john to recover the archive password demonstrating that "encryption" on consumer archive tools is often only as strong as the password behind it.

6\. Custom Rules <br/>

explored john's rule-based manglng (defined in john.conf) to generate password variants (e.g., appending numbers or capitalizing letters) increasing crack success rate against human password patterns.

<h2>Findings</h2>

- several hashes were cracked within seconds using a standard wordlist, reinforcing how quickly weak or reused passwords fall to common tools.

- hash type misidentification was the most common source of failed cracking attempts because correct format selection matters as much as wordlist quality.

- archive and ssh key passwords, while often overlooked, are just as vulnerable to offline cracking as traditional account hashes if the underlying password is weak.

<h2>Defensive Takeaways</h2>

- hashing algorithm matters: md5/sha-1 are fast to crack at scale. modern systems should use slow, salted algorithms (bcrypt, scrypt, etc.) to resist offline attacks.

- password policy: length and complexity requirements reduce dictionary attack success far more than periodic rotation policies do.

- detection opportunity: while this lab focused on offline cracking (generates no network or log signal), the same credential weaknesses are what enable online attacks like credential stuffing and password spraying (e.g., windows event id 4625, ssh auth failures).

- recommendations: a siem correlation rule alerting on n failed logins across multiple accounts from a single source in a short window would help catch spraying attempts stemming from a breached password list.
