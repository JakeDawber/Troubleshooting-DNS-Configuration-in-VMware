<h1>Troubleshooting DNS configuration in VMware</h1>

<b>1. Identifying the Problem</b>

During the setup of my Active Directory home lab, I encountered a DNS configuration issue while installing and configuring the DNS Server role. The Add Roles and Features Wizard displayed a warning stating that no static IP address had been detected on the server. This highlighted the importance of assigning a static IP address to a domain controller to ensure that clients could reliably locate DNS and Active Directory services.


<img src="Screenshots/DNS error.png"/>


<b>2. Investigating the Network Configuration</b>

I used the following command to inspect the server's current network settings:

ipconfig /all

The output showed that the server was using a dynamically assigned IP address, with DHCP enabled and DNS settings provided by the VMware virtual network. This configuration was unsuitable for a domain controller that needed a consistent network address.

<img src="Screenshots/Ip config.png"/>


I also inspected the VMware Workstation Virtual Network Editor to understand how DHCP was configured for the host-only network

The network used the following configuration:

Subnet: 192.168.193.0/24

Subnet mask: 255.255.255.0

DHCP address pool: 192.168.193.128 – 192.168.193.254

Reviewing the DHCP range helped me understand how VMware was allocating addresses and how to choose a suitable static IP address outside the automatically assigned pool.

<b>3. Configuring a Static IP Address</b>

I configured the Windows Server network adapter to use a static IPv4 address rather than relying on DHCP.

The server was assigned the following settings:


IPv4 address - 192.168.193.10

Subnet mask - 255.255.255.0

Default gateway - Left blank for the isolated host-only network

Preferred DNS server - 192.168.193.10

The server's IP address was outside VMware's DHCP allocation range, reducing the risk of an address conflict. I configured the server to use its own static IP address as its preferred DNS server so that it could resolve records hosted by its local DNS service.

<b>4. Testing and Verifying DNS</b>

After configuring the static IP address, I continued troubleshooting using Command Prompt and PowerShell.

I used nslookup to test DNS resolution and inspected the DNS Server service status using:

sc query dns

I also checked whether the DNS server was listening on port 53:

netstat -ano | findstr ":53"

The output showed DNS-related listeners on the server's IPv4 address, including 192.168.193.10:53.

To investigate further, I used PowerShell's Resolve-DnsName cmdlet to query the DNS server directly:

Resolve-DnsName jh-dc01.jakehomelab.test -Server 192.168.193.10 -Type A

This successfully returned 192.168.193.10. I also tested the local DNS resolver and verified the server's forward and reverse DNS records. Finally, I ran:

dcdiag /test:dns

The diagnostic output reported that the server passed the DNS test and that the jakehomelab.test domain passed its DNS test.

Although some nslookup commands continued to display timeout warnings, the successful PowerShell queries and DNS diagnostic results demonstrated that the relevant DNS records were resolving correctly. The remaining timeout behaviour indicated that further investigation of DNS query responses was still warranted.

Skills Demonstrated

Troubleshooting Windows Server network configuration.

Understanding DHCP allocation ranges and static IP addressing.

Configuring DNS settings on an Active Directory domain controller.

Investigating DNS service status and port 53 listeners.

Testing forward and reverse DNS resolution.

Using Command Prompt and PowerShell troubleshooting tools.

Running Active Directory DNS diagnostics with dcdiag.


<h2>Process</h2>

DNS Service Verification: Used netstat to confirm that DNS is listening on TCP and UDP port 53 on the domain controller's assigned IPv4 address. Successful DNS resolution and Active Directory DNS diagnostics were verified separately.

<img src="Screenshots/Ip config.png"/>

Active Directory DNS health: Used dcdiag to verify Active Directory DNS health and from the output, we can see that the Windows server JH-dc01 successfully passes both the connectivity test and the DNS test. We can also see that the domain jakehomelab.test successfully passed the DNS test.

<img src="Screenshots/AD health.png"/>
