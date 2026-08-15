# Password Hashing and Storage
Student Name: Mohammed Omer Ahmned

Student ID:  12303667

Project: Password-Hashing-12303667

# 1. Overview
This activity focused on comparing different password hashing algorithms and improving authentication security using PAM policies. MD5-crypt, SHA-512 and yescrypt were examined, followed by password-quality testing, account lockout and password-cracking comparisons.

# 2. Network Setup
A GNS3 project was created with one Ubuntu Host labelled **Target**.

*Evidence:
<img width="1919" height="1019" alt="1 network week 3" src="https://github.com/user-attachments/assets/9cbf0b44-ed8a-4568-8224-942c255d928f" />


# 3. Password Hash Algorithms

Three users were created using the same student-ID-based password but different hashing algorithms:

-  `omer_md5` — MD5-crypt
- `omer_sha512` — SHA-512
- `omer_yescrypt` — yescrypt

The entries were examined using:

```bash
cat /etc/shadow | grep omer_
```

The prefixes identify the hashing algorithms:

| Prefix | Algorithm |
| ------ | --------- |
| `$1$`  | MD5-crypt |
| `$6$`  | SHA-512   |
| `$y$`  | yescrypt  |

**Evidence:** 
<img width="1899" height="1017" alt="2 network week 3" src="https://github.com/user-attachments/assets/ef82bbc3-aa00-4241-b9b0-4835329073d5" />


## 4. PAM Password Policies

`pam_pwquality` was configured to require a minimum password length of 12 characters, at least one uppercase letter and at least one digit.

```text
password requisite pam_pwquality.so retry=3 minlen=12 ucredit=-1 dcredit=-1 enforce_for_root dictcheck=0
```

A separate `user_test` account was used for testing. A weak password such as `abc` was rejected, while a password meeting the requirements was accepted.

The account lockout policy was also configured:

```text
deny = 5
unlock_time = 300
```

After five failed authentication attempts, the account was locked for 300 seconds. The lockout status was checked using:

```bash
faillock --user user_test
```
**Evidence:**
<img width="1912" height="1026" alt="3 network week 3" src="https://github.com/user-attachments/assets/3848bc5f-3ab8-4977-b105-cf02bbcc5527" />

## 5. Password Cracking

A second set of accounts was created with a password confirmed to exist in `rockyou.txt`. This allowed the hashing algorithms to be tested using a password cracker.

John the Ripper was used to perform the dictionary attack.

```bash
grep '^Tahsin_' /etc/shadow > hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

The testing demonstrated that MD5-crypt was faster to attack, while SHA-512 was slower and yescrypt was designed to make password cracking more computationally expensive.

**Evidence:** 
<img width="1894" height="1020" alt="4 network week 3" src="https://github.com/user-attachments/assets/149628f8-5236-4ad2-b8b2-991594517c15" />


# Analysis Questions

## 1. What do `$1$`, `$6$` and `$y$` mean?

The prefixes identify the password hashing algorithm used in `/etc/shadow`. `$1$` represents MD5-crypt, `$6$` represents SHA-512 and `$y$` represents yescrypt. The algorithm matters because stronger and slower algorithms make large-scale password guessing more expensive for attackers.

## 2. Why are slow hashing algorithms preferred?

Algorithms such as yescrypt are intentionally computationally expensive. Although this adds some processing time during legitimate authentication, it greatly increases the time and resources required for an attacker to test large numbers of password guesses.

## 3. What is a salt?

A salt is a random value combined with a password before hashing. It helps prevent attackers from efficiently using precomputed hash tables and also means identical passwords can produce different stored hashes.

In `/etc/shadow`, the salt appears after the algorithm identifier and before the following `$`. The exact salt should be identified from the hash generated during the activity.

## 4. What is a disadvantage of `pam_faillock`?

A disadvantage of account lockout is that it can be abused for denial-of-service. An attacker could intentionally submit incorrect passwords repeatedly for a known username, causing the legitimate user's account to become temporarily locked.

## Conclusion

This activity demonstrated the importance of secure password storage and authentication controls. Comparing MD5-crypt, SHA-512 and yescrypt showed why the choice of hashing algorithm matters, while PAM password-quality and account-lockout policies provided additional protection against weak passwords and repeated login attempts.
