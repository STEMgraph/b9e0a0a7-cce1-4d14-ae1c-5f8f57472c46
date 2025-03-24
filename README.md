<!---
{
  "depends_on": ["SSH"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-03-17",
  "keywords": ["encryption", "SSH", "RSA"]
}
--->

# Using asymmetrical key pairs for SSH

## 1) Introduction
In 1976, Whitfield Diffie and Martin Hellman introduced a groundbreaking concept that transformed cryptography forever: asymmetric cryptography. Their pioneering work, presented in the seminal paper "New Directions in Cryptography," laid the foundation for secure communications across the digital world. Today, public-key encryption is an essential security standard used in numerous applications, including secure web browsing (HTTPS), encrypted emails, and remote server connections.

In this exercise, you'll leverage asymmetric cryptography to secure your SSH connections. Instead of relying on a conventional password typed into a command prompt—which can be vulnerable to brute-force attacks and human error—you will create a unique pair of cryptographic keys. These keys enable you to securely authenticate and encrypt messages exchanged between your client and server, significantly enhancing the security and reliability of your remote interactions.

<details>
  <summary>Server and Client</summary>
  Server and Client are functional names given to two partners of a communication. The server is already running and is just waiting for incoming connections. The client on the other hand can connect and use a service at any time and disconnect again. Often times there will be just one server, providing a service and multiple clients connecting to the same server. To learn more about the [client-server-architecture check out the linked exercise](www.github.com/STEMgraph/missing).
</details>

Without public-key encryption, establishing an SSH connection involves the client initiating a request to the server. Once the session is initiated, the server prompts the client for a password, which authenticates the user's identity by confirming that the user knows a secret known only to them. However, this method's security depends heavily on password management by users. Long, complex passwords are secure but difficult for individuals to remember, often leading to the use of shorter, weaker passwords. The number of robust passwords that an average user can effectively remember is limited. By utilizing cryptographic keys securely stored on your device—which itself is protected by a strong password—you mitigate these risks and strengthen your overall security posture.

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
<DONT SHOW ANYONE YOUR PRIVATE KEYS!>
r2ID1XoyGp+G7VUlILDMAAAAFW1heGNsZXJrd2VsbEBsaWFvZG9uZw==
-----END OPENSSH PRIVATE KEY-----
```

For public-key-encryption, we create such a _private key_, as seen above.
It is always generated in a pair of _private_ and _public_ key. 
When the client now sends a message to the server, the client calculates a specific number which is called HASH, uses a mathematical operation to _sign_ this number, and attaches it to the message. 
The server now receives two parts of the message:

1. The message itself
2. A number that is a result of a mathematical operation that encapsules a property of the message and a secret of the user. 

<details>
  <summary>HASH-Functions</summary>
  Have you ever calculated the digit-sum of a multi-digit number? This is a contracting mapping. HASH-functions work somewhat similar. Their purpose is to map a long string of numbers onto a smaller set. This is used in error-detection algorithms, compression, password-storage, encryption and multiple other places. To learn more about [HASH-functions chek out the linked exercise](www.github.com/STEMgraph/missing).
</details>

The server receives the message but halts before using it, since it wants to check the clients identity. 
First it calculates the HASH-value of the message itself. 
Since the sender signed the value with their private key, these values don't match. 
This is where the second part of the key - the public-key - comes into play.
The public key needs to be known to the server in order to successfully authenticate the clients signature.
In a mathematical operation, the public key is applied to the signed HASH-value. 
The result of this operation: The unencrypted HASH-value. 
Now the server can compare its value to the signed one.

This establishes two levels of trust:

1. The message was not manipulated after the sender calculated the HASH
2. The message was definitly sent by the owner of that private key

Everybody with the matching public key can ensure, that these two properties are true. 
When the server sends a message back to the client, the same methode is applied, but with the servers private key.


### 1.1) Further Readings and Other Sources
- [Computerphile about Public Key Cryptographie](https://www.youtube.com/watch?v=GSIDS_lvRv4)
- Barker, Chen, et.al.. (2019). [Recommendation for Pair-Wise Key Establishment Using Integer Factorization Cryptography](https://doi.org/10.6028/NIST.SP.800-56Br2)
- Diffie and Hellman. (1976). [New directions in cryptography](https://doi.org/10.1109/TIT.1976.1055638)
- Shannon. (1949). ["Communication Theory of Secrecy Systems"](https://doi.org/10.1002/j.1538-7305.1949.tb00928.x)


## 2) Tasks
1. **Creating a Key Pair**: Run `ls` (or `DIR` on Windows) to see the list of all files in your current directory. Open your terminal and type `ssh-keygen`. You will be confronted with a dialog, that will generate a pair of keys. When promted for a password, leave the input empty and press `Return`.
2. **Inspecting the Keys**: Run `ls` (or `DIR` on Windows) and find the generated key-pair. Use `cat` on Linux and Mac, or `type` on Windows to inspect the content of the files.
3. **Setting a passphrase**: Run `ssh-keygen` again, but now with the additional parameter `-f <filename_for_new_key>`. Replace the placeholder with a name like `protected_key`. Now, use a passphrase while generating it. Passphrases significantly improve security, as a compromised key file without passphrase protection could grant immediate access to attackers.
4. **Compare the Keys**: Inspect the newly generated key-pair and compare them to one another.

<details>
  <summary>VIM</summary>
  If installed, use `vim -O` to display files side-by-side. To get more familiar with [vim`, check out the link](www.github.com/STEMgraph/missing).
</details>

5. **Clean up!**: Delete both key pairs from your current directory. Access your users `home` directory. Check whether you can access the `.ssh` subdirectory, which may or may not be located in your `home` directory. If it doesn't exist, create it with `mkdir .ssh`. Make sure, that you don't forget the `.` in front of the name. `cd` into it.
6. **Generate a new Key Pair**: Now generate a new key pair with some additional properties:

```sh
ssh-keygen -t ed25519 -f <filename_for_your_key> -C "This key is used for machine ...."
```

With the additional parameter `-t` the key-generator is configured to generate a specific type of key: `ed25519`. 
Since we want to be able to use multiple keys for multiple remote-machines, you can use `-C` to attach the remote-machines name to the key.
Decide for yourself, whether you want to give your key a passphrase or not.
7. **Copy the .pub Key**: In order for the server to verify our signature, the `<filename_of_the_key.pub` part of our key pair needs to be copied to the remote machine. You can either use `ssh-copy-id` for that, or do it manually. For the sake of understanding what is going on behind the curtain, let's do it manually:
1) Print your `.pub` key to your terminal
2) Copy it to your clipboard
3) [SSH into your remote machine](https://github.com/STEMgraph/8c79cd1f-f6bd-4930-b62c-b2970c412735)
4) Navigate to `~/.ssh`
5) Open the file `authorized_keys` with a texteditor. If it doesn't exist yet, create it!
6) Copy your `.pub` key into the file.
7) Save and log out of the SSH session.
8. **Logging in with Key Pair**: Make sure, that your terminal is displaying your local machine. Retype the SSH command, but this time give it an additional parameter `-i` for _Identity_:

```sh
ssh -i ~/.ssh/<filename_of_your_key> <username>@<host>
```

## 3) Questions
1. What is the advantage of asymmetric cryptography compared to symmetric cryptography?
2. Why should private keys never be shared?
3. Explain in your own words how a server verifies a client’s signature using asymmetric cryptography.
4. If someone gets access to your public key, can they decrypt your messages? Why or why not?
5. Why is it advisable to use different keys for different services or servers?

## 4) Advice
If you want to dive more into security-engineering, set up a virtual machine for yourself and play around with _KALI_-Linux. 
Maybe start with this exercise: [LINK!](www.github.com/STEMgraph/missing)
