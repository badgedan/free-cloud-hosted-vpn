
# Creating A *FREE* Private VPN Using OpenVPN

Each one of us has their own opinion on privacy over the internet, so for people who are in some way overthink a lot like me, it's not easy to trust every privacy regulation you see.

So, I've conducted a **TOTALLY** free VPN connection by utilizing both [Oracle's Cloud Free Tier](https://www.oracle.com/cloud/free/) and [OpenVPN Access Server](https://openvpn.net/as-docs/general.html), surely this has some limits but having your own data traveled securely via a some-way trusted intermediary is better than handing your data to a "Free" VPN provider.

## Architecture Overview

![arch](https://i.imgur.com/Invj7Tg.png)

I'm not going to discuss how VPNs work, but having a high-level look at how this architecture may look like would give us a greater understanding of how to do setup it up correctly.

Our VM instance that's hosted on the cloud would work as the intermediary and be enabled via the OpenVPN protocol.

## Setting Up The Environment

Create your own account on [Oracle Cloud](https://signup.cloud.oracle.com/), sadly you will need a credit card to be able to create an account.![cloud-register](https://i.imgur.com/e2PHdtK.png)

After creating your own account you can create a VM instance, specifically the free tier eligible ones, there're two options here either the **ARM CPU Based**, or **AMD CPU Based**. Each has its own use cases but due to the high demand on the **ARM CPU Based**, we're creating an **AMD CPU Based** instance.  
  
  

This is going to be our go-to, this is enough to handle a simple VPN server, just don't forget to download the SSH private key.

![shape](https://i.imgur.com/U9XvsLO.png)

This is going to be our instance after launching, you can see the Public IP address which we can copy and SSH to the server.

![instance-dashboard](https://i.imgur.com/177oe0d.png)

After accessing the VM through SSH
![mobaxtreamssh](https://i.imgur.com/XSbmthC.png)

We can use the following set of commands to install OpenVPN access Server

```
sudo su
apt install ca-certificates wget net-tools gnupg
wget -qO /etc/apt/trusted.gpg.d/openvpn-as-repo.gpg https://as-repository.openvpn.net/as-repo-public.gpg
```

Type the command `lsb_release -a` to get the codename of your OS

**CHANGE THE CODENAME**
```
echo "deb http://as-repository.openvpn.net/as/debian {CODENAME} main">/etc/apt/sources.list.d/openvpn-as-repo.list
apt update
```
Now we install OpenVPN access Server
`apt install openvpn-as`

After installing you'll get an output having both username and password, store them in a notepad or something, as we can already configure the default password via the `passwd openvpn` command or the Admin UI but just to make sure.

![ssh-install](https://i.imgur.com/rGKlwWv.png)


Oke! We successfully installed our own OpenVPN Access Server, just a few adjustments before going all in, you can see I've highlighted both HTTPS (**443**) and **943** which are used for hosting the Access Server. 

To allow them in the firewall we need to go to our instances, then click on the VCN
![vcn](https://i.imgur.com/YcDfrBG.png)

Then we get to see our subnet which we'll allow the specified ports on
![subnet](https://i.imgur.com/S2o7bZ6.png)

Click on the security list
![seclist](https://i.imgur.com/yrvDu7m.png)

Click on add **Add Ingress Rules** and specify the ports we've adjusted, also source CIDR to enable it on all private IP addresses.
![ingressrules](https://i.imgur.com/IGwdQyJ.png)

OKAY! NOW WE'RE REALLY DONE WITH THE INSTANCE SETUP, now we head to set up the OpenVPN Access Server :3

going to **[https://\[YOUR_INSTANCE_PUBLIC_IP_ADDRESS\]:943/admin](https://%5BYOUR_INSTANCE_PUBLIC_IP_ADDRESS%5D:943/admin)** would show us a lot of things, for the purpose of this tutorial we need to adjust only 3 things. 

1) Create an account used to login to the OpenVPN server
2) Change the IP address that's used to connect to the VPN server
3) Add DNS configuration

Going to the User Management tab, then to User Permissions, you create an account and don't forget to add a password

![user](https://i.imgur.com/70psX9W.png)

Going to the Configuration tab, then to Network Settings, you'll see the IP address that clients use to connect to your VPN server.

![netsettings](https://i.imgur.com/phMSTSt.png)

Also, you should apply the load-balancing option in the protocol sector, if you live in a country such as Egypt, OpenVPN on UDP connections is blocked by the ISP, but if you live in a country which UDP is enabled you should allow it as you would remove the overhead that TCP and OpenVPN packet retransmitting techniques create. Also if you want to read more about how to bypass the WireGuard block in Egypt check out this [article](https://www.ntkernel.com/how-to-bypass-egypts-wireguard-ban/) 

Enough of this, on to the final step and let's add our DNS resolver. In the same tab as the Network Settings, click on VPN Settings, after scrolling you'll find the DNS sector, enabling clients to choose their DNS servers, and then add your preferred DNS resolver.

![dns](https://i.imgur.com/zf7B95z.png)


## Connecting Via OpenVPN Software on Windows

Create a profile with your VM's public IP address
![ovpn1](https://i.imgur.com/Jw0E8Sq.png)
Then specify the username and password that we've created before.
![ovpn2](https://i.imgur.com/EsWtvUg.png)


And voilà we're **CONNECTED**!

![connection](https://i.imgur.com/wHBiRlP.png)

## Things To Consider

If you check the logs, in Egypt the UDP connection will fail but it'll work when it's a TCP connection, this is mainly because as we've said before that Egypt blocks OpenVPN UDP based connections. So as this is a TCP connection you might see some delays, a workaround is to use Stunnel in combination with OpenVPN UDP-based.

![FAIL](https://i.imgur.com/8GtzdAA.png)![sccss](https://i.imgur.com/2bJ902r.png)

You can see the difference in latency in these screenshots.

**Before connecting to our VPN** This is how fast our ICMP packets in the send and reply process.
![ping](https://i.imgur.com/OJwhJ2T.png)

**After connecting to our VPN** This is how fast our ICMP packets in the send and reply process.
![latency](https://i.imgur.com/SWJAWf2.png)


