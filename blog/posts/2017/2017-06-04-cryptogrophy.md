# Modern Cryptography (and how to crack it?)

### NOTE - EVERYTHING IN THIS ARTICLE IS MEANT FOR EDUCATIONAL PURPOSES ONLY.

"The enciphering and deciphering of messages in secret code or cipher; also :  the computerized encoding and decoding of information" - [Merriam-Webster Dictionary](https://www.merriam-webster.com/dictionary/cryptography). Enigma, hashing, and the Caesar Cipher are all examples of cryptogrophy. 

Cryptography can be applied to military communities, electronic commerce, ATM cards, computer passwords, etc. Cryptography used to be synonymous with encryption, but modern more complex, mathematics theory based cryptography sets encryption and cryptogrophy apart. Such cryptographic algorithms generate theoretically non-collidable complex multi-character strings that can be used to make sure files aren't modified during transfer as files/text/whatever always generate the same 'hash'.

Ex. If we SHA-1 hash "Cryptography" we get:

    b804ec5a0d83d19d8db908572f51196505d09f98
    
and if we do it again, we get the same thing (even when ran through a different program with the same algorithm): 

    b804ec5a0d83d19d8db908572f51196505d09f98
   
but "hello" generates: 

    aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
    
so how can these extremly complex 'hashes' be broken?

Simply put, collisions (and hashcat). Hashcat is meant to be used as a password recovery tool for when a user forgets their original password but has the hash of said password. Hashcat works through various methods including, exploiting flaws in hashing algorithms as they are found by it's creator, Jens 'atom' Steube, and other methods in cluding, Brute-Force, Dictionary Attacks, and many more. The main goal of hashcat is to run through combinations of alphanumeric/Unicode characters until the hash matches the original inputted hash. 
### Hashcat requires a large amount of processing power so it is reccomended that you do not try to use hashcat unless you have access to *very* high-end hardware (eg. Nvidia Quadro/1080(ti) and Intel Core i7/Xeon).

Before we go any further, what hashing algorithms are supported by hashcat?

    MD4
    MD5
    Half MD5 (left, mid, right)
    SHA1
    SHA-224
    SHA-256
    SHA-384
    SHA-512
    SHA-3 (Keccak)
    SipHash
    Skip32
    RIPEMD-160
    Whirlpool
    DES (PT = $salt, key = $pass)
    3DES (PT = $salt, key = $pass)
    GOST R 34.11-94
    GOST R 34.11-2012 (Streebog) 256-bit
    GOST R 34.11-2012 (Streebog) 512-bit
    md5($pass.$salt)
    md5($salt.$pass)
    md5(unicode($pass).$salt)
    md5($salt.unicode($pass))
    md5($salt.$pass.$salt)
    md5($salt.md5($pass))
    md5($salt.md5($salt.$pass))
    md5($salt.md5($pass.$salt))
    md5(md5($pass))
    md5(md5($pass).md5($salt))
    md5(strtoupper(md5($pass)))
    md5(sha1($pass))
    sha1($pass.$salt)
    sha1($salt.$pass)
    sha1(unicode($pass).$salt)
    sha1($salt.unicode($pass))
    sha1(sha1($pass))
    sha1($salt.sha1($pass))
    sha1(md5($pass))
    sha1($salt.$pass.$salt)
    sha1(CX)
    sha256($pass.$salt)
    sha256($salt.$pass)
    sha256(unicode($pass).$salt)
    sha256($salt.unicode($pass))
    sha512($pass.$salt)
    sha512($salt.$pass)
    sha512(unicode($pass).$salt)
    sha512($salt.unicode($pass))
    HMAC-MD5 (key = $pass)
    HMAC-MD5 (key = $salt)
    HMAC-SHA1 (key = $pass)
    HMAC-SHA1 (key = $salt)
    HMAC-SHA256 (key = $pass)
    HMAC-SHA256 (key = $salt)
    HMAC-SHA512 (key = $pass)
    HMAC-SHA512 (key = $salt)
    PBKDF2-HMAC-MD5
    PBKDF2-HMAC-SHA1
    PBKDF2-HMAC-SHA256
    PBKDF2-HMAC-SHA512
    MyBB
    phpBB3
    SMF (Simple Machines Forum)
    vBulletin
    IPB (Invision Power Board)
    WBB (Woltlab Burning Board)
    osCommerce
    xt:Commerce
    PrestaShop
    MediaWiki B type
    WordPress
    Drupal 7
    Joomla
    PHPS
    Django (SHA-1)
    Django (PBKDF2-SHA256)
    Episerver
    ColdFusion 10+
    Apache MD5-APR
    MySQL
    PostgreSQL
    MSSQL
    Oracle H: Type (Oracle 7+)
    Oracle S: Type (Oracle 11+)
    Oracle T: Type (Oracle 12+)
    Sybase
    hMailServer
    DNSSEC (NSEC3)
    IKE-PSK
    IPMI2 RAKP
    iSCSI CHAP
    CRAM-MD5
    MySQL CRAM (SHA1)
    PostgreSQL CRAM (MD5)
    SIP digest authentication (MD5)
    WPA
    WPA2
    NetNTLMv1
    NetNTLMv1+ESS
    NetNTLMv2
    Kerberos 5 AS-REQ Pre-Auth etype 23
    Kerberos 5 TGS-REP etype 23
    Netscape LDAP SHA/SSHA
    FileZilla Server
    LM
    NTLM
    Domain Cached Credentials (DCC), MS Cache
    Domain Cached Credentials 2 (DCC2), MS Cache 2
    MS-AzureSync PBKDF2-HMAC-SHA256
    descrypt
    bsdicrypt
    md5crypt
    sha256crypt
    sha512crypt
    bcrypt
    scrypt
    OSX v10.4
    OSX v10.5
    OSX v10.6
    OSX v10.7
    OSX v10.8
    OSX v10.9
    OSX v10.10
    iTunes backup < 10.0
    iTunes backup >= 10.0
    AIX {smd5}
    AIX {ssha1}
    AIX {ssha256}
    AIX {ssha512}
    Cisco-ASA MD5
    Cisco-PIX MD5
    Cisco-IOS $1$ (MD5)
    Cisco-IOS type 4 (SHA256)
    Cisco $8$ (PBKDF2-SHA256)
    Cisco $9$ (scrypt)
    Juniper IVE
    Juniper NetScreen/SSG (ScreenOS)
    Juniper/NetBSD sha1crypt
    Fortigate (FortiOS)
    Samsung Android Password/PIN
    Windows Phone 8+ PIN/password
    GRUB 2
    CRC32
    RACF
    Radmin2
    Redmine
    PunBB
    OpenCart
    Atlassian (PBKDF2-HMAC-SHA1)
    Citrix NetScaler
    SAP CODVN B (BCODE)
    SAP CODVN F/G (PASSCODE)
    SAP CODVN H (PWDSALTEDHASH) iSSHA-1
    PeopleSoft
    PeopleSoft PS_TOKEN
    Skype
    WinZip
    7-Zip
    RAR3-hp
    RAR5
    AxCrypt
    AxCrypt in-memory SHA1
    PDF 1.1 - 1.3 (Acrobat 2 - 4)
    PDF 1.4 - 1.6 (Acrobat 5 - 8)
    PDF 1.7 Level 3 (Acrobat 9)
    PDF 1.7 Level 8 (Acrobat 10 - 11)
    MS Office <= 2003 MD5
    MS Office <= 2003 SHA1
    MS Office 2007
    MS Office 2010
    MS Office 2013
    Lotus Notes/Domino 5
    Lotus Notes/Domino 6
    Lotus Notes/Domino 8
    Bitcoin/Litecoin wallet.dat
    Blockchain, My Wallet
    1Password, agilekeychain
    1Password, cloudkeychain
    LastPass
    Password Safe v2
    Password Safe v3
    KeePass 1 (AES/Twofish) and KeePass 2 (AES)
    eCryptfs
    Android FDE <= 4.3
    Android FDE (Samsung DEK)
    TrueCrypt
    VeraCrypt
    LUKS
    Plaintext

Now let's go over some of the many ways hashcat can attempt to crack your hash. <br>
______________________________________________________________________________________________________

| Attack                             | What The Attack Does                                                         |
|------------------------------------|------------------------------------------------------------------------------|
| Brute-Force Attack and Mask Attack | Trying all characters from given charsets, per position                      |
| Dictionary Attack                  | Trying all words in a list; also called “straight” mode                      |
| Combinator Attack                  | Concatenating words from multiple wordlists                                  |
| Hybrid Attack                      | Combining wordlists+masks and masks+wordlists                                |
| Rule-based Attack                  | Applying rules to words from wordlists; combines with wordlist-based attacks |
| Toggle-case Attack                 | Toggling case of characters                                                  |

## In conclusion

Modern cryptogrophy is complex, yet powerful, *but* it does have it's flaws as shown with hashcat. Luckily, cryptogrophy is hard to crack even with hashcat, and as methods grow more and more complex it will only become harder.

## References
[Hashcat](https://hashcat.net/hashcat/) <br>
[Hashcat - Wikipedia](https://en.wikipedia.org/wiki/Hashcat)<br>
[Cryptography - Wikipedia](https://en.wikipedia.org/wiki/Cryptography)<br>
[Hash Function - Wikipedia](https://en.wikipedia.org/wiki/Hash_function)<br>
[Cryptographic Hash Function - Wikipedia](https://en.wikipedia.org/wiki/Cryptographic_hash_function)<br>
[cryptii](https://cryptii.com/text/sha1)




