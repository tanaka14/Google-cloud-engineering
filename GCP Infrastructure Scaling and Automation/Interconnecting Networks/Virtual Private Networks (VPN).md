![Diagram](img/cloud1.png)

## Overview

Here We establish VPN tunnels between two networks in separate regions such that a VM in one network can ping a VM in the other network over its internal IP address.

## OBjectives

The objective is to know how to perform the following tasks:

- Create VPN gateways in each network

- Create VPN tunnels between the gateways

- Verify VPN connectivity



## Task 1:  Explore or Create the networks and instances

Two custom networks with VM instances have been configured for you. if not create them, For the purposes of the lab, both networks are VPC networks within a Google Cloud project. However, in a real-world application, one of these networks might be in a different Google Cloud project, on-premises, or in a different cloud.


# - Explore the networks

Verify that vpn-network-1 and vpn-network-2 have been created with subnets in separate regions.

1. In the Cloud Console, on the Navigation menu (Navigation menu), click VPC network > VPC networks.

    - Note the vpn-network-1 network and its subnet-a in us-central1.

    - Note the vpn-network-2 network and its subnet-b in europe-west1.


# - Explore the firewall rules

1. In the navigation pane, click Firewall.

    - Note the network-1-allow-ssh and network-1-allow-icmp rules for vpn-network-1.
    - Note the network-2-allow-ssh and network-2-allow-icmp rules for vpn-network-2.

These firewall rules allow SSH and ICMP traffic from anywhere.

# - Explore the instances and their connectivity

Currently, the VPN connection between the two networks is not established. Explore the connectivity options between the instances in the networks.

1. In the Cloud Console, on the Navigation menu (Navigation menu), click Compute Engine > VM instances.
2. Click Columns, and select Network.
3. Note the external and internal IP addresses for server-2.
4. For server-1, click SSH to launch a terminal and connect.

5. To test connectivity to server-2's external IP address, run the following command, replacing server-2's external IP address with the value noted earlier:


Code -> [ ping -c 3 <Enter server-2's external IP address here> ]

*This works because the VM instances can communicate over the internet.

6. To test connectivity to server-2's internal IP address, run the following command, replacing server-2's internal IP address with the value noted earlier:

Code -> [ ping -c 3 <Enter server-2's internal IP address here> ]

*You should see 100% packet loss when pinging the internal IP address because you don't have VPN connectivity yet.

7. Exit the SSH terminal.
    Let's try the same from server-2.

8. Note the external and internal IP addresses for server-1.

9. For server-2, click SSH to launch a terminal and connect.

10. To test connectivity to server-1's external IP address, run the following command, replacing server-1's external IP address with the value noted earlier:

Code -> [ ping -c 3 <Enter server-1's external IP address here> ]

11. To test connectivity to server-1's internal IP address, run the following command, replacing server-1's internal IP address with the value noted earlier:

Code -> [ ping -c 3 <Enter server-1's internal IP address here> ]

*You should see similar results.

12. Exit the SSH terminal.

*Why are we testing both server-1 to server-2 and server-2 to server-1?

For the purposes of this lab, the path from subnet-a to subnet-b is not the same as the path from subnet-b to subnet-a. You are using one tunnel to pass traffic in each direction. And if both tunnels are not established, you won't be able to ping the remote server on its internal IP address. The ping might reach the remote server, but the response can't be returned.

This makes it much easier to debug the lab during class. In practice, a single tunnel could be used with symmetric configuration. However, it is more common to have multiple tunnels or multiple gateways and VPNs for production work, because a single tunnel could be a single point of failure.





## Task 2:  Create the VPN gateways and tunnels

Establish private communication between the two VM instances by creating VPN gateways and tunnels between the two networks.

# - Reserve two static IP addresses

Reserve one static IP address for each VPN gateway.


1. In the Cloud Console, on the Navigation menu (Navigation menu), click VPC network > External IP addresses.

    - Note the vpn-network-1 network and its subnet-a in us-central1.

    - Note the vpn-network-2 network and its subnet-b in europe-west1.


2. Click *Reserve static address.

3. Specify the following, and leave the remaining settings as their defaults:


Property	Value (type value or select option as specified)
Name	vpn-1-static-ip
IP version	IPv4
Region	us-central1


4. Click Reserve.

- Repeat the same for vpn-2-static-ip.

5. Click Reserve static address.

6. Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	vpn-2-static-ip
IP version	IPv4
Region	europe-west1

7. Click Reserve.

Note both IP addresses for the next step. They will be referred to us [VPN-1-STATIC-IP] and [VPN-2-STATIC-IP].

## Create the vpn-1 gateway and tunnel1to2

1. In the Cloud Console, on the Navigation menu (Navigation menu), click Hybrid Connectivity > VPN.

2. Click Create VPN Connection.

3. If asked, select Classic VPN, and then click Continue.

4. Specify the following in the VPN gateway section, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	vpn-1
Network	vpn-network-1
Region	us-central1
IP address	vpn-1-static-ip


5. Specify the following in the Tunnels section, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	tunnel1to2
Remote peer IP address	[VPN-2-STATIC-IP]
IKE pre-shared key	gcprocks
Routing options	Route-based
Remote network IP ranges	10.1.3.0/24

Make sure to replace [VPN-2-STATIC-IP] with your reserved IP address for europe-west1.

6. Click command line.
  *The gcloud command line window shows the gcloud commands to create the VPN gateway and VPN tunnels and it illustrates that three forwarding rules are also created.

7. Click Close.
8. Click Create.



Create the vpn-2 gateway and tunnel2to1

1. Click VPN setup wizard.

2. If asked, select Classic VPN, and then click Continue.

3. Specify the following in the VPN gateway section, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	vpn-2
Network	vpn-network-2
Region	europe-west1
IP address	vpn-2-static-ip

4. Specify the following in the Tunnels section, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	tunnel2to1
Remote peer IP address	[VPN-1-STATIC-IP]
IKE pre-shared key	gcprocks
Routing options	Route-based
Remote network IP ranges	10.5.4.0/24

Make sure to replace [VPN-1-STATIC-IP] with your reserved IP address for us-central1.


5. Click Create.
6. Click Cloud VPN Tunnels.


Wait for the VPN tunnels status to change to Established for both tunnels before continuing.



## Task 3: Verify VPN connectivity

# -Verify server-1 to server-2 connectivity


1. In the Cloud Console, on the Navigation menu, click Compute Engine > VM instances.

2. For server-1, click SSH to launch a terminal and connect.

3. To test connectivity to server-2's internal IP address, run the following command:

Code -> [ ping -c 3 <insert server-2's internal IP address here> ]

4. Exit the server-1 SSH terminal.

5. For server-2, click SSH to launch a terminal and connect.

6. To test connectivity to server-1's internal IP address, run the following command:

Code -> [ ping -c 3 <insert server-1's internal IP address here> ]


## Remove the external IP addresses

Now that you verified VPN connectivity, you can remove the instances' external IP addresses. For demonstration purposes, just do this for the server-1 instance.

1. On the Navigation menu, click Compute Engine > VM instances.
2. Select the server-1 instance and click Stop. Wait for the instance to stop.
    *Instances need to be stopped before you can make changes to their network interfaces.
3. Click on the name of the server-1 instance to open the VM instance details page.
4. Click Edit.
5. For Network interfaces, click the Edit icon (Edit).
6. Change External IP to None.
7. Click Done.
8. Click Save and wait for the instance details to update.
9. Click Start.
10. Click Start again to confirm that you want to start the VM instance.
11. Return to the VM instances page and wait for the instance to start.
12. Notice that External IP is set to None for the server-1 instance.
        *Feel free to SSH to server-2 and verify that you can still ping the server-1 instance's internal IP address. You still be able to SSH to server-1 from the Cloud Console using Cloud IAP as described here.

    *External IP addresses that don’t fall under the Free Tier will incur a small cost. Also, as a general security best practice, it’s a good idea to use internal IP addresses where applicable and since you configured Cloud VPN you no longer need to communicate between instances using their external IP address.

## Task 4: Review

In this lab, you configured a VPN connection between two networks with subnets in different regions. Then you verified the VPN connection by pinging VMs in different networks using their internal IP addresses.

You configured the VPN gateways and tunnels using the Cloud Console. However, this approach obfuscated the creation of forwarding rules, which you explored with the command line button in the Console. This can help in troubleshooting a configuration if needed.