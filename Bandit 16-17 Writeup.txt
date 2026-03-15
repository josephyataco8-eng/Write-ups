# Bandit Level 16 → 17 Write-up

Platform: OverTheWire  
Challenge: Bandit Level 16 → 17

---

## Objective

Retrieve the password for the next level by identifying the correct service running on a specific port and authenticating with the obtained credentials.

---

## 1. Port Scanning

First, we scan the ports in the range **31000–32000** using `nmap`.

```bash
nmap -sT -p 31000-32000 localhost
```

### Options

- **-sT** : TCP connect scan.  
  This scan is used because we do not have special privileges in this environment.  
  Unlike SYN scans, `-sT` works without root privileges.

- **-p** : specifies the port range.

### Output (relevant part)

```
Not shown: 996 closed tcp ports (conn-refused)

PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown
```

Several ports are open, so the next step is identifying the running services.

---

## 2. Service Detection

We run another `nmap` scan to determine the service versions.

```bash
nmap -sV -p 31046,31518,31691,31790,31960 localhost
```

### Options

- **-sV** : enables service/version detection.  
  This allows us to identify which services are running and determine whether any of them use **SSL**.

- **-p** : specifies the ports to scan.

### Output (relevant part)

```
PORT      STATE SERVICE     VERSION
31046/tcp open  echo
31518/tcp open  ssl/echo
31691/tcp open  echo
31790/tcp open  ssl/unknown
31960/tcp open  echo
```

The scan shows that **ports 31518 and 31790 provide SSL services**.

---

## 3. Connecting to the SSL Service

We attempt to connect to the SSL services using **OpenSSL**.

### Attempt 1

```bash
printf "password\n" | openssl s_client -connect localhost:31518 -quiet
```

### Attempt 2

```bash
printf "password\n" | openssl s_client -connect localhost:31790 -quiet
```

### Command Explanation

- **printf** : prints the password and sends it to the next command.
- **| (pipe)** : passes the output of one command as input to another.
- **openssl s_client** : makes OpenSSL act as an SSL/TLS client.
- **-connect** : specifies the host and port.
- **-quiet** : suppresses unnecessary output.

### Results

- **Port 31518** → No useful output.  
- **Port 31790** → Returns an **RSA private key**.

Example output:

```
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

This private key can be used for **SSH authentication**.

---

## 4. Preparing the Private Key

SSH requires that private key files have strict permissions.  
Otherwise, SSH will refuse to use them for security reasons.

Assuming the key is saved as:

```
credenciales_RSA
```

We change the permissions:

```bash
chmod 600 credenciales_RSA
```

This ensures that only the owner can **read and write** the file.

---

## 5. SSH Authentication

Now we authenticate using the obtained private key.

```bash
ssh bandit17@bandit.labs.overthewire.org -i credenciales_RSA -p 2220
```

### Options

- **-i** : specifies the identity file (private key).
- **-p** : specifies the SSH port.

---

## Result

Successful login to **Bandit Level 17**.

```
bandit17@bandit:~$
```

