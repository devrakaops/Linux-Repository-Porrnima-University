
---

# Linux Service Management & Server Management — Complete Understanding

Let's start from zero.

Imagine you have a Linux machine.

You want this machine to provide some service to other machines.

For example:

- You want to provide a website → **Web Server**
- You want remote login → **SSH Server**
- You want database access → **Database Server**
- You want file sharing → **NFS Server**

So the first question is:

> **What makes a Linux machine a server?**

---

# 1. What is a Server?

A server is not necessarily a special type of computer.

A normal Linux machine can become a server if it provides some service to other systems.

For example:

```text
Linux Machine
     |
     +---- Provides Web Service
     |          ↓
     |      Web Server
     |
     +---- Provides SSH Service
     |          ↓
     |      SSH Server
     |
     +---- Provides Database Service
                ↓
           Database Server
```

So, very simply:

> **Server = A system that provides a service to clients.**

For example, if your Linux machine provides a website, we call it a **Web Server**.

If the same Linux machine provides SSH access, it is also an **SSH Server**.

This is important:

### One Linux machine can be multiple servers at the same time.

For example:

```text
                 Linux Machine
                       |
        +--------------+--------------+
        |              |              |
        ↓              ↓              ↓
    Web Server     SSH Server    Database Server
```

There is nothing wrong with this.

---

# 2. But How Does the Server Keep Running?

Now imagine we have a Web Server.

A client sends:

```text
"Give me your website."
```

The request comes to the Linux machine.

Something must receive that request and process it.

So Apache must be running continuously.

For example:

```text
Client
   |
   | HTTP Request
   ↓
Linux Server
   |
   ↓
Apache
   |
   ↓
Website Response
```

Now imagine Apache runs only once:

```text
Apache starts
    ↓
Does something
    ↓
Stops
```

Then a client sends a request five minutes later.

Who will answer?

Nobody.

Therefore, server software normally needs a **long-running background process**.

And this brings us to the concept of a **daemon**.

---

# 3. What is a Daemon?

A **daemon** is a background process that runs continuously and performs a service-related task.

For example:

```text
httpd
sshd
mariadbd
```

These are examples of daemon processes.

Think of it this way:

```text
Normal command:

User runs command
      ↓
Command does its job
      ↓
Command exits
```

But a daemon behaves more like:

```text
Daemon starts
     ↓
Runs in background
     ↓
Waits for requests
     ↓
Processes requests
     ↓
Continues running
     ↓
Waits for next request
```

For a Web Server:

```text
httpd
  |
  +---- waits for HTTP requests
  |
  +---- receives request
  |
  +---- processes request
  |
  +---- sends response
  |
  +---- waits again
```

That's why a web server needs a continuously running background process.

---

# 4. Then What is a Service?

This is where beginners often get confused.

People commonly say:

> "Start the Apache service."

But technically, there are different concepts involved.

We have:

```text
Apache Software
       ↓
httpd daemon/process
       ↓
systemd service unit
       ↓
systemctl command
```

A **systemd service unit** describes how systemd should manage a service.

For Apache, the service unit is:

```text
httpd.service
```

The actual Apache process is:

```text
httpd
```

So when we execute:

```bash
systemctl start httpd
```

we are asking **systemd** to start and manage the Apache service.

---

# 5. Why Do We Need systemd?

Now imagine you have 50 different services.

You want Linux to:

- Start services during boot
- Stop services
- Restart services
- Monitor services
- Start services in the correct order
- Automatically restart some failed services
- Manage service dependencies
- Control which services start automatically

Doing all of this manually would be difficult.

Therefore, Linux uses a service manager.

On modern RHEL-based systems, that service manager is:

> **systemd**

And we communicate with systemd using:

> **systemctl**

So:

```text
systemd
   ↑
   |
systemctl
```

---

# 6. systemctl — Our Main Service Management Tool

Now the commands make sense.

### Start

```bash
systemctl start httpd
```

Meaning:

> Tell systemd to start the Apache service.

### Stop

```bash
systemctl stop httpd
```

Meaning:

> Tell systemd to stop Apache.

### Restart

```bash
systemctl restart httpd
```

Meaning:

> Stop and start the service again.

### Status

```bash
systemctl status httpd
```

Meaning:

> Tell me the current state of Apache.

### Enable

```bash
systemctl enable httpd
```

Meaning:

> Start Apache automatically during system boot.

### Disable

```bash
systemctl disable httpd
```

Meaning:

> Do not automatically start Apache during boot.

### Enable + Start

```bash
systemctl enable --now httpd
```

Meaning:

> Start it now and also start it automatically after future boots.

---

# 7. Start vs Enable — Very Important

This is one of the most common beginner mistakes.

Suppose:

```bash
systemctl start httpd
```

Apache starts.

Then you reboot the machine.

Apache may not start automatically.

Why?

Because:

```text
start
```

means:

> Start it **right now**.

Whereas:

```bash
systemctl enable httpd
```

means:

> Start it automatically during **future boot**.

Therefore:

```text
start  → Now
enable → Boot
```

And:

```bash
systemctl enable --now httpd
```

means:

```text
Start now
   +
Start automatically after reboot
```

---

# 8. Now We Have a Server

Let's say we installed Apache.

```bash
yum install httpd
```

Now we have:

```text
Package
   ↓
httpd
```

We start it:

```bash
systemctl start httpd
```

Now Apache is running.

But is the server accessible from the network?

**Not necessarily.**

This is the next important concept.

---

# 9. How Does a Client Find the Correct Server?

Suppose your Linux machine has this IP:

```text
192.168.1.100
```

A client wants to access your website.

It sends:

```text
192.168.1.100
```

But wait.

You told me earlier that the same machine can run:

- Web Server
- SSH Server
- Database Server
- NFS Server

So if a request comes to:

```text
192.168.1.100
```

how does Linux know which service should receive it?

This is where **ports** become necessary.

---

# 10. What is a Port?

Think about an IP address as the **address of a building**.

And think about a port as the **door of a particular service**.

For example:

```text
192.168.1.100:80
```

means:

```text
Machine → 192.168.1.100
Service endpoint → Port 80
```

Another request:

```text
192.168.1.100:22
```

goes to the SSH service.

Another:

```text
192.168.1.100:3306
```

can go to MySQL/MariaDB.

So:

```text
                192.168.1.100
                      |
          +-----------+-----------+
          |           |           |
          ↓           ↓           ↓
       :80          :22        :3306
          |           |           |
       Apache        SSH       Database
```

This is why ports are required.

---

# 11. Common Ports

| Service | Common Port |
|---|---:|
| SSH | 22/TCP |
| HTTP | 80/TCP |
| HTTPS | 443/TCP |
| MySQL/MariaDB | 3306/TCP |
| NFS | 2049 |

So for a Web Server:

```text
IP Address + Port
```

For example:

```text
192.168.1.100:80
```

---

# 12. The Four Things We Should Know About Any Server

Now we can introduce a very useful learning model.

Whenever you work with a server, ask four questions:

```text
1. What package provides it?
2. What service manages it?
3. Which port does it use?
4. Where is its configuration?
```

For Apache:

| Item | Apache |
|---|---|
| Package | `httpd` |
| Service | `httpd.service` |
| Port | `80/TCP`, `443/TCP` |
| Main configuration | `/etc/httpd/conf/httpd.conf` |

For SSH:

| Item | SSH |
|---|---|
| Package | `openssh-server` |
| Service | `sshd.service` |
| Port | `22/TCP` |
| Configuration | `/etc/ssh/sshd_config` |

For MariaDB:

| Item | MariaDB |
|---|---|
| Package | MariaDB package |
| Service | `mariadb.service` |
| Port | `3306/TCP` |
| Configuration | `/etc/my.cnf` and related files |

For NFS:

| Item | NFS |
|---|---|
| Package | `nfs-utils` |
| Service | `nfs-server.service` |
| Port | `2049` |
| Configuration | `/etc/exports` |

This is an excellent troubleshooting model.

---

# 13. Let's Build Our Web Server

Now let's start the actual practical.

We want:

```text
Linux Machine
     ↓
Web Server
     ↓
Apache
```

First, we need the package.

```bash
yum install httpd
```

Check:

```bash
rpm -q httpd
```

Now start Apache:

```bash
systemctl start httpd
```

Check:

```bash
systemctl status httpd
```

We want:

```text
Active: active (running)
```

---

# 14. Where Does Apache Get the Website From?

Apache needs some content to serve.

The default document root is normally:

```text
/var/www/html/
```

So let's create:

```bash
vim /var/www/html/index.html
```

For example:

```html
<html>
<head>
    <title>My Web Server</title>
</head>
<body>
    <h1>Hello, this is my website</h1>
    <h2>I am running this web server</h2>
</body>
</html>
```

Now Apache has something to serve.

---

# 15. Test It Locally

Before involving the network, let's first ask:

> Is Apache itself working?

Run:

```bash
curl http://localhost
```

If everything is working, Apache should return the HTML content.

We can also use:

```bash
curl http://127.0.0.1
```

This is a very important troubleshooting habit.

Don't immediately test from another machine.

First test:

```text
Application itself
       ↓
Local machine
       ↓
Network
       ↓
Remote machine
```

This helps us identify where the problem exists.

---

# 16. But Can Another Machine Access It?

Suppose your server is:

```text
192.168.1.100
```

Another machine tries:

```text
http://192.168.1.100
```

The request reaches the server's network interface.

But suppose it doesn't work.

Now we have an important question:

> **Apache is running. Why can't another machine access it?**

This is exactly where the **firewall** becomes necessary.

---

# 17. Why Do We Need a Firewall?

Think about the server from a security perspective.

Your Linux machine may have many services running:

```text
22    → SSH
80    → HTTP
443   → HTTPS
3306  → Database
2049  → NFS
```

Do you really want every network connection to every service to be accepted?

Usually, **no**.

For example, maybe you want:

```text
SSH → Allowed
HTTP → Allowed
Database → Not allowed from the network
NFS → Not allowed from outside
```

So we need something that can control network traffic.

That is the role of the:

> **Firewall**

---

# 18. What Does a Firewall Actually Do?

A firewall applies rules to network traffic.

For example:

```text
Client
   |
   | TCP/80
   ↓
Firewall
   |
   | Rule says HTTP is allowed
   ↓
Apache
```

But:

```text
Client
   |
   | TCP/3306
   ↓
Firewall
   |
   | Rule says Database is not allowed
   X
```

So the firewall answers a question like:

> **Should this network traffic be allowed or blocked?**

This is why the firewall becomes necessary **after** we understand ports and network access.

---

# 19. Firewall and Apache Are Different Things

This distinction is extremely important.

Suppose:

```text
Apache → Running
```

and:

```text
Port 80 → Listening
```

but:

```text
Firewall → Blocking TCP/80
```

Then:

```text
Local access
       ↓
May work

Remote access
       ↓
May fail
```

So:

> **A running service does not automatically mean that the service is reachable from the network.**

There are multiple layers.

---

# 20. Check Whether Apache Is Listening

Use:

```bash
ss -lntp
```

Or:

```bash
ss -lntp | grep :80
```

You may see:

```text
LISTEN ... :80 ... httpd
```

This tells us:

> Apache is listening on TCP port 80.

But this still does not prove that another machine can reach it.

Why?

Because the firewall may still block it.

---

# 21. Allow HTTP Through Firewall

We can allow HTTP using the predefined service:

```bash
firewall-cmd --permanent --add-service=http
```

Then:

```bash
firewall-cmd --reload
```

Check:

```bash
firewall-cmd --list-all
```

Now another machine can try:

```text
http://192.168.1.100
```

---

# 22. Port vs Firewall — Don't Mix Them Up

Suppose Apache is listening on:

```text
80
```

That means:

> Apache is ready to receive traffic on port 80.

Firewall allowing port 80 means:

> The network is allowed to reach port 80.

These are different things.

```text
Apache
  ↓
Listening on 80
  ↓
Firewall
  ↓
Allows 80
  ↓
Remote client can reach Apache
```

---

# 23. Now Let's Change the Port

Suppose your requirement is:

> "I don't want Apache to use port 80. I want it to use port 82."

Open:

```bash
vim /etc/httpd/conf/httpd.conf
```

Find:

```text
Listen 80
```

Change:

```text
Listen 82
```

Now test the configuration:

```bash
apachectl configtest
```

If:

```text
Syntax OK
```

then:

```bash
systemctl restart httpd
```

---

# 24. But What If Apache Doesn't Start?

This is where good troubleshooting starts.

We changed the port from:

```text
80
```

to:

```text
82
```

And Apache fails to restart.

A beginner may think:

> "I changed the port, so I should just open port 82 in the firewall."

But wait.

The firewall controls **network traffic**.

Apache may not even have started.

So first check:

```bash
systemctl status httpd
```

And:

```bash
journalctl -u httpd
```

Now we might discover that another security layer is stopping Apache.

That layer is:

> **SELinux**

---

# 25. Why Do We Need SELinux?

Now the requirement for SELinux becomes easy to understand.

We already have:

```text
Application
   ↓
Apache
   ↓
Firewall
```

But imagine an attacker somehow compromises Apache.

Should Apache automatically be allowed to do anything on the system?

No.

For example, if Apache is compromised, we don't want it to automatically:

- Access every file
- Read sensitive system data
- Use arbitrary resources
- Perform unauthorized actions
- Bind to arbitrary ports

So Linux needs another security layer that controls **what a process is allowed to do**.

This is where SELinux comes in.

---

# 26. Firewall vs SELinux — The Big Difference

This is one of the most important concepts.

### Firewall asks:

> **Can this network traffic reach the machine/service?**

### SELinux asks:

> **Is this process allowed to perform this action according to the security policy?**

For example:

```text
Client
   |
   | TCP/82
   ↓
Firewall
   |
   | "Is TCP/82 allowed?"
   ↓
SELinux
   |
   | "Is httpd allowed to use TCP/82?"
   ↓
Apache
```

So:

```text
Firewall → Network-level control

SELinux → Mandatory access control / security policy
```

They are not replacements for each other.

They solve different problems.

---

# 27. What is SELinux?

SELinux means:

> **Security-Enhanced Linux**

It provides an additional security layer based on security policies and labels/contexts.

On RHEL systems, SELinux is an important part of the security model.

A simplified view is:

```text
Normal Linux permissions
          +
       SELinux
          ↓
Additional security control
```

---

# 28. SELinux Modes

SELinux has three modes:

```text
Enforcing
Permissive
Disabled
```

Let's understand them properly.

---

## Enforcing

SELinux is active and enforcing its policies.

If an action is not allowed:

```text
Application
     |
     | Unauthorized action
     ↓
 SELinux
     |
     X
 Block
```

This is the normal security mode.

---

## Permissive

SELinux still checks and reports policy violations, but does not block the action.

Think:

```text
Action
  ↓
SELinux
  ↓
"Policy violation detected"
  ↓
Log it
  ↓
Do not block
```

This is very useful for troubleshooting.

---

## Disabled

SELinux is disabled.

There is no SELinux enforcement or normal SELinux policy checking.

For production systems, disabling SELinux should not be treated as the default solution to an SELinux problem.

---

# 29. Check SELinux Mode

Run:

```bash
getenforce
```

You might get:

```text
Enforcing
```

or:

```text
Permissive
```

or:

```text
Disabled
```

---

# 30. Temporarily Change SELinux Mode

You can temporarily switch:

```bash
setenforce 0
```

This changes to:

```text
Permissive
```

And:

```bash
setenforce 1
```

changes to:

```text
Enforcing
```

Check:

```bash
getenforce
```

Important:

> `setenforce` changes the runtime state. It does not permanently modify the configuration.

---

# 31. Permanent SELinux Configuration

The configuration file is:

```text
/etc/selinux/config
```

For example:

```text
SELINUX=enforcing
```

or:

```text
SELINUX=permissive
```

or:

```text
SELINUX=disabled
```

So remember:

```text
setenforce
     ↓
Temporary runtime change

/etc/selinux/config
     ↓
Persistent configuration
```

---

# 32. Why Does SELinux Stop Our Port 82?

Now come back to our practical.

Apache normally uses:

```text
80
443
```

SELinux already knows that these ports are valid for HTTP services.

But we suddenly tell Apache:

```text
"Start using 82."
```

SELinux asks:

> "Is TCP/82 an approved HTTP port?"

If the answer is:

> No.

SELinux can prevent Apache from using that port.

This is a **security feature**, not a bug.

The idea is:

```text
Apache
   |
   | Wants TCP/82
   ↓
SELinux
   |
   | "Is 82 allowed for HTTP?"
   |
   +---- NO → Block
```

---

# 33. SELinux Uses Labels/Types

SELinux associates security types/labels with resources.

For HTTP ports, an important type is:

```text
http_port_t
```

You can see SELinux port mappings using:

```bash
semanage port -l
```

To filter HTTP-related entries:

```bash
semanage port -l | grep http
```

You may see:

```text
http_port_t
```

The important idea is:

> SELinux recognizes certain ports as valid ports for HTTP services.

---

# 34. Check Port 82

Run:

```bash
semanage port -l | grep 82
```

If port 82 is not mapped to the appropriate HTTP port type, Apache may not be allowed to use it while SELinux is enforcing.

---

# 35. We Have Two Possible Approaches

Now we have a choice.

### Approach 1 — Disable/relax SELinux

For example, temporarily:

```bash
setenforce 0
```

Apache may then be able to use port 82.

But this is **not the preferred production solution**.

Why?

Because we are weakening the security layer instead of correctly configuring it.

---

### Approach 2 — Correctly Configure SELinux

This is the better approach.

Tell SELinux:

> "Port 82 is also an HTTP port."

Use:

```bash
semanage port -a -t http_port_t -p tcp 82
```

Let's understand this command.

```text
semanage port
```

Manage SELinux port mappings.

```text
-a
```

Add.

```text
-t http_port_t
```

Assign the HTTP port type.

```text
-p tcp
```

Protocol is TCP.

```text
82
```

Port number.

So:

```bash
semanage port -a -t http_port_t -p tcp 82
```

means:

> Add TCP port 82 to the SELinux HTTP port type.

---

# 36. Verify the SELinux Configuration

Run:

```bash
semanage port -l | grep 82
```

Now you should see that port 82 is associated with:

```text
http_port_t
```

Now SELinux understands:

> TCP/82 is allowed for HTTP services.

---

# 37. Don't Forget the Firewall

This is another important point.

We fixed SELinux.

But remember:

```text
SELinux ≠ Firewall
```

We still need to allow TCP/82 through the firewall.

```bash
firewall-cmd --permanent --add-port=82/tcp
```

Then:

```bash
firewall-cmd --reload
```

Check:

```bash
firewall-cmd --list-all
```

Now both security layers are configured.

```text
Firewall
   ↓
TCP/82 allowed

SELinux
   ↓
httpd allowed to use TCP/82
```

---

# 38. Restart Apache

Now:

```bash
systemctl restart httpd
```

Check:

```bash
systemctl status httpd
```

Then:

```bash
ss -lntp | grep :82
```

We want to see Apache listening on:

```text
82
```

---

# 39. Access the Website on Port 82

Because port 82 is not the default HTTP port, we must specify it.

From the server:

```bash
curl http://localhost:82
```

From another machine:

```text
http://192.168.1.100:82
```

Notice:

```text
:82
```

Why?

Because the browser normally assumes:

```text
http → 80
https → 443
```

But we changed our Apache server to:

```text
82
```

So we explicitly tell the client:

> "Connect to port 82."

---

# 40. Now the Entire Story Makes Sense

Let's put everything together.

We started with:

> I want to make my Linux machine a Web Server.

### Step 1 — We need Apache

So we install the package:

```bash
yum install httpd
```

### Step 2 — Apache must continuously run

So we start its service:

```bash
systemctl start httpd
```

### Step 3 — We don't want to start it manually after every reboot

So:

```bash
systemctl enable httpd
```

### Step 4 — Apache needs a network endpoint

So it listens on:

```text
TCP/80
```

### Step 5 — We need website content

So Apache serves:

```text
/var/www/html/
```

### Step 6 — Local testing works

```bash
curl http://localhost
```

### Step 7 — Another machine wants to access it

Now network traffic comes from outside.

We ask:

> Should outside traffic be allowed?

This creates the need for:

**Firewall**

```bash
firewall-cmd --permanent --add-service=http
```

### Step 8 — We change Apache from 80 to 82

Now Apache wants:

```text
TCP/82
```

But SELinux says:

> "I know HTTP ports such as 80 and 443, but I don't know 82 as an HTTP port."

This creates the need to understand:

**SELinux**

### Step 9 — Configure SELinux

```bash
semanage port -a -t http_port_t -p tcp 82
```

### Step 10 — Configure firewall

```bash
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload
```

### Step 11 — Restart Apache

```bash
systemctl restart httpd
```

### Step 12 — Test

```bash
curl http://localhost:82
```

From another machine:

```text
http://192.168.1.100:82
```

Now the entire learning journey is connected.

---

# 41. The Most Important Mental Model

Don't memorize these as independent topics:

```text
systemctl
port
firewall
SELinux
httpd
configuration
```

Instead, think of one continuous story:

```text
                 I want to provide a service
                            |
                            ↓
                         SERVER
                            |
                            ↓
                  Install the software
                            |
                            ↓
                         PACKAGE
                            |
                            ↓
                  Run it continuously
                            |
                            ↓
                         DAEMON
                            |
                            ↓
                    Managed by systemd
                            |
                            ↓
                         SERVICE
                            |
                            ↓
                    Listen on a PORT
                            |
                            ↓
                  Client wants access
                            |
                            ↓
                    NETWORK TRAFFIC
                            |
                            ↓
                       FIREWALL
                            |
                  "Should I allow it?"
                            |
                            ↓
                        SELINUX
                            |
                  "Is this action allowed
                    for this process?"
                            |
                            ↓
                       APPLICATION
                            |
                            ↓
                    WEB CONTENT
                            |
                            ↓
                         CLIENT
```

That is the complete concept.

---

# 42. Server Management Mind Map

```text
                         SERVER
                           |
        +------------------+------------------+
        |                  |                  |
     Package             Service             Port
        |                  |                  |
     httpd            httpd.service          80
        |                  |                  |
        |              systemd            443
        |                  |                  |
        |              systemctl              |
        |                  |                  |
        +------------------+------------------+
                           |
                    Configuration
                           |
                /etc/httpd/conf/httpd.conf
                           |
                           ↓
                     Network Access
                           |
                           ↓
                       Firewall
                           |
                           ↓
                        SELinux
                           |
                           ↓
                       Web Server
                           |
                           ↓
                    /var/www/html/
                           |
                           ↓
                         Client
```

---

# 43. A Very Important Troubleshooting Mindset

Suppose someone says:

> **"My website is not working."**

Don't immediately run random commands.

Walk through the architecture.

### Question 1

Is the package installed?

```bash
rpm -q httpd
```

### Question 2

Is the service running?

```bash
systemctl status httpd
```

### Question 3

Is Apache listening?

```bash
ss -lntp | grep :80
```

### Question 4

Is the configuration valid?

```bash
apachectl configtest
```

### Question 5

Is the firewall allowing the port?

```bash
firewall-cmd --list-all
```

### Question 6

Is SELinux enforcing?

```bash
getenforce
```

### Question 7

If using a custom port, does SELinux allow it?

```bash
semanage port -l | grep http
```

### Question 8

Does the content exist?

```bash
ls -l /var/www/html/
```

### Question 9

Does local access work?

```bash
curl http://localhost
```

### Question 10

Does remote access work?

```text
http://SERVER_IP
```

This gives you a structured troubleshooting approach instead of guessing.

---

# 44. One Final Difference You Should Remember

There are **three different questions** here:

### Question 1 — Is the application running?

```bash
systemctl status httpd
```

### Question 2 — Is the application listening?

```bash
ss -lntp | grep :80
```

### Question 3 — Can another machine reach it?

Check:

```text
Firewall
Network
SELinux
Routing
```

These are not the same question.

For example:

```text
httpd running        → YES
httpd listening 80   → YES
remote access        → NO
```

In this situation, Apache itself may be completely healthy.

The problem could be:

```text
Firewall
Network
SELinux
Routing
```

---

# 45. Final Picture

If I had to teach this entire topic in one picture, I would explain it like this:

```text
                    LINUX SERVER
                         |
                         |
             "I want to provide a service"
                         |
                         ↓
                    APPLICATION
                         |
                         ↓
                      DAEMON
                  (background process)
                         |
                         ↓
                    systemd
                         |
                         ↓
                  httpd.service
                         |
                         ↓
                     systemctl
                         |
                         ↓
                  Listen on Port
                         |
                    TCP/80 or 82
                         |
                         ↓
               +-------------------+
               |      FIREWALL     |
               |                   |
               | Allow / Block     |
               | network traffic   |
               +---------+---------+
                         |
                         ↓
               +-------------------+
               |      SELINUX      |
               |                   |
               | Is httpd allowed  |
               | to use this port? |
               +---------+---------+
                         |
                         ↓
                    HTTP SERVER
                         |
                         ↓
                  DocumentRoot
                         |
                         ↓
                  /var/www/html/
                         |
                         ↓
                     index.html
                         |
                         ↓
                       CLIENT
```

The key idea is not to memorize **"firewall command"** or **"SELinux command"** separately.

Understand **why they appeared**:

> **First we needed a server → the server needed a continuously running daemon → the daemon needed management → systemd/systemctl came in → the server needed a network port → clients needed network access → firewall became necessary → we needed stronger process-level security → SELinux became important.**

Once you understand this chain, the commands become much easier to remember because **every command has a reason behind it**.
