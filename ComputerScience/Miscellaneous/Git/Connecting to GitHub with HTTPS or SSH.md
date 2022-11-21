GitHub no longer accept account passwords when authenticating Git operations on GitHub.com since August 13, 2021, at 09:00 PST. Instead, token-based authentication (for example, personal access, OAuth, SSH Key, or GitHub App installation token) will be required for all authenticated Git operations.

Nowadays GitHub suggest using a Personal Access Token (works with HTTPS) or SSH key (works with SSH) for git authentication.

## HTTPS

If you want to connecting to GitHub with HTTPS, use PAT (Personal Access Token) instead of password. Create a PAT is simple, please refer to the [documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

## SSH

SSH (Secure Shell Protocol) can be used to connect and authenticate ro remote servers and services, include GitHub. The advantage of SSH keys is that you can connect to GitHub without supplying your username and personal access token at each visit. You can also use an SSH key to sign commits.

### How does SSH work

SSH applications are based on a **client–server** architecture. SSH operates on TCP port 22 by default. The host (server) listens on port 22 (or any other SSH assigned port) for incoming connections. When the a client tries to connect to the server via TCP:

1. _**Negotiate session encryption**_. The server and client swap the encryption protocols and respective versions that they support, find a matched one. The the _connection is started with the accepted protocol._ The server also uses an _asymmetric public key_ which the client can use to verify the authenticity of the host. Hashing algorithm such as MD5 (RSA key fingerprint in SSH) can be used to defense against man-in-the-middle attacks while SSH doesn't have CA like HTTPS protocol.
2. **_Authenticate the user_**, using password encrypted by the previous agreed symmetric algorithm or the **SSH Key Pair**, which is more secure and convenient, also the one we use in connecting to GitHub, technical details will be explained in next section.
3. _If the above verification is successful_, opening the correct shell environment. Then we can _**transmit the data**_ encrypted by the previous agreed symmetric algorithm.

_Connection setup_ and _authentication_ of SSH use asymmetric and hasing algorithm, which ensure security while _data transmission_ of SSH use symmetric algorithm, which ensure the efficiency of communication.

### Connect with SSH

On Unix-like systems, the list of authorized public keys is typically stored in the home directory of the user that is allowed to log in remotely, in the file _~/.ssh/authorized_keys._ This file is respected by SSH **only if it is not writable by anything apart from the owner and root**.

**1.** Enter the _~/.ssh_ directory and list the files to check for existing SSH keys.

```shell
cd ~/.ssh && ls
```

By default, the filenames of supported public keys for GitHub are one of the following:

- *id_rsa.pub*
- *id_ecdsa.pub*
- *id_ed25519.pub*
    
If any of them exists, select one, jump to step **4** 👉

**2.** If none of above public keys exists, generating a new SSH key. Replace the email address with your GitHub email:

```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

If you are using a legacy system that doesn't support the Ed25519 algorithm, use rsa:

```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This creates a new SSH key, using the provided email as a label.

**3.** Add your public key to the `ssh-agent`. `ssh-agent` is a key manager for SSH. It holds your keys and certificates in memory, unencrypted, and ready for use by `ssh`.

First start the `ssh-agent` in the background:

```shell
eval "$(ssh-agent -s)"
```

Then add your SSH **private key** to the `ssh-agent`. Replace it if you created your key with a different name:

```shell
ssh-add ~/.ssh/id_rsa
```

**4.** Add the new SSH key to GitHub account. Create a new SSH key on your GitHub account, title it and copy the **public key** to the key content:

```shell
cat id_rsa.pub
```

Now you can test if your SSH key works:

```shell
ssh -T git@github.com 
```

If success, you will see the following prompt:

> Hi <yourname>! You've successfully authenticated, but GitHub does not provide shell access.

Now you can use `git` to access GitHub repositories throught SSH without offering PAT each time. Each time client tries to connect to the server with SSH, at the Authentication stage, server will send a random string encrypted by the SSH public key you upload before (and then encrypted by the symmetric key agreed at negotiation stage), the client will (first use the symmetric key to decrypt the data, then) decrypt the content by SSH private key, and send it back (this time encrypt by private key, decrypt by public key). If the data are the same, the authentication is successful.