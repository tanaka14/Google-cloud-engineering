![Diagram](img/cloud1.png)

# Overview

![GoDoc](https://godoc.org/github.com/moby/buildkit?status.svg)
![Build Status](https://github.com/moby/buildkit/workflows/build/badge.svg)
![Go Report Card](https://goreportcard.com/badge/github.com/moby/buildkit)



Google Cloud HTTP(S) load balancing is implemented at the edge of Google's network in Google's points of presence (POP) around the world. User traffic directed to an HTTP(S) load balancer enters the POP closest to the user and is then load-balanced over Google's global network to the closest backend that has sufficient available capacity.

In this lab, you configure an HTTP load balancer as shown in the diagram below. Then, you stress test the load balancer to demonstrate global load balancing and autoscaling.


![Diagram](img/cloud1.png)


## Objectives

The objective is to know how to perform the following tasks:

- Create a health check firewall rule
- Create a NAT configuration using Cloud Router
- Create a custom image for a web server
- Create an instance template based on the custom image
- Create two managed instance groups
- Configure an HTTP load balancer with IPv4 and IPv6
- Stress test an HTTP load balancer



## Task 1:  Configure a health check firewall rule

Health checks determine which instances of a load balancer can receive new connections. For HTTP load balancing, the health check probes to your load-balanced instances come from addresses in the ranges `130.211.0.0/22 and 35.191.0.0/16`. Your firewall rules must allow these connections.


#### - Create the health check rule

Create a firewall rule to allow health checks.


1. In the Cloud Console, on the `Navigation menu` (Navigation menu), click `VPC network > Firewall`. Notice the existing `ICMP, internal, RDP,` and `SSH` firewall rules. 

`Each Google Cloud project starts with the default network and these firewall rules.   `

2. Click Create Firewall Rule.

3. Specify the following, and leave the remaining settings as their defaults:

| Property  | Value (type value or select option as specified) |
| ------------- | ------------- |
| Name | fw-allow-health-checks  |
| Targets  | Specified target tags  |
| Target tags  | allow-health-checks  |
| Source filter  | IP Ranges  |
| Source IP ranges  | 130.211.0.0/22 and 35.191.0.0/16  |
| Protocols and ports  | Specified protocols and ports  |


#### - Explore the firewall rules

1. In the navigation pane, click Firewall.

    - Note the network-1-allow-ssh and network-1-allow-icmp rules for vpn-network-1.
    - Note the network-2-allow-ssh and network-2-allow-icmp rules for vpn-network-2.

Make sure to include the /22 and /16 in the Source IP ranges.

Select tcp and specify port 80.
Click Create.
Click Check my progress to verify the objective.
Configure health check firewall rule

Task 2: Create a NAT configuration using Cloud Router
The Google Cloud VM backend instances that you setup in Task 3 will not be configured with external IP addresses.

Instead, you will setup the Cloud NAT service to allow these VM instances to send outbound traffic only through the Cloud NAT, and receive inbound traffic through the load balancer.

Create the Cloud Router instance
In the Cloud Console, on the Navigation menu (Navigation menu), click Network services > Cloud NAT.

Click Get started.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Gateway name	nat-config
Network	default
Region	us-central1
Click Cloud Router, and select Create new router.

For Name, type nat-router-us-central1.

Click Create.

In Create a NAT gateway, click Create.

Wait until the NAT Gateway Status changes to Running before moving onto the next task.

Click Check my progress to verify the objective.
Create a NAT configuration using Cloud Router

Task 3: Create a custom image for a web server
Create a custom web server image for the backend of the load balancer.

Create a VM
In the Cloud Console, on the Navigation menu (Navigation menu), click Compute Engine > VM instances.

Click Create Instance.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	webserver
Region	us-central1
Zone	us-central1-a
Series	N1
Machine type	f1-micro (1 vCPU)
Under Boot disk, select change.

click Show advanced option.

Under deletion rule, select keep boot disk

Click Select

Click Networking, disks, security, management, sole-tenancy.

Click Networking.

For Network tags, type allow-health-checks.
Under Network interfaces , click default.
Under External IP dropdown, select None.
Click Done.

Click Create.

Customize the VM
For webserver, click SSH to launch a terminal and connect.

If you see the Connection via Cloud Identity-Aware Proxy Failed popup, click Retry.

To install Apache2, run the following commands:

sudo apt-get update
sudo apt-get install -y apache2
Copied!
To start the Apache server, run the following command:

sudo service apache2 start
Copied!
To test the default page for the Apache2 server, run the following command:

curl localhost
Copied!
The default page for the Apache2 server should be displayed.

Set the Apache service to start at boot
The software installation was successful. However, when a new VM is created using this image, the freshly booted VM does not have the Apache web server running. Use the following command to set the Apache service to automatically start on boot. Then test it to make sure it works.

In the webserver SSH terminal, set the service to start on boot:

sudo update-rc.d apache2 enable
Copied!
In the Cloud Console, select webserver, and then click Reset.
In the confirmation dialog, click Reset.
Reset will stop and reboot the machine. It keeps the same IPs and the same persistent boot disk, but memory is wiped. Therefore, if the Apache service is available after the reset, the update-rc command was successful.

Check the server by connecting via SSH to the VM and entering the following command:

sudo service apache2 status
Copied!
NOTE: If you see the Connection via Cloud Identity-Aware Proxy Failed popup, click Retry .
The result should show Started The Apache HTTP Server.

Prepare the disk to create a custom image
Verify that the boot disk will not be deleted when the instance is deleted.

On the VM instances page, click webserver to view the VM instance details.

Under Boot disk, verify that When deleting instance is set to Keep disk.

Return to the VM instances page, click webserver, and click Delete.

In the confirmation dialog, click Delete.

In the left pane, click Disks and verify that the webserver disk exists.

Create the custom image
In the left pane, click Images.

Click Create image.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	mywebserver
Source	Disk
Source disk	webserver
Click Create.

You have created a custom image that multiple identical webservers can be started from. At this point, you could delete the webserver disk.

The next step is to use that image to define an instance template that can be used in the managed instance groups.

Click Check my progress to verify the objective.
Create a custom image for a web server

Task 4. Configure an instance template and create instance groups
A managed instance group uses an instance template to create a group of identical instances. Use these to create the backends of the HTTP load balancer.

Configure the instance template
An instance template is an API resource that you can use to create VM instances and managed instance groups. Instance templates define the machine type, boot disk image, subnet, labels, and other instance properties.

In the Cloud Console, on the Navigation menu (Navigation menu), click Compute Engine > Instance templates.

Click Create instance template.

For Name, type mywebserver-template.

For Series, select N1.

For Machine type, select f1-micro (1 vCPU).

For Boot disk, click Change.

Click Custom images.

For Image, Select mywebserver.

Click Select.

Click Management, security, disks, networking, sole tenancy.

Click Networking.

For Network tags, type allow-health-checks.
Under External IP dropdown, select None.
Click Create.

Create the managed instance groups
Create a managed instance group in us-central1 and one in europe-west1.

On the Navigation menu, click Compute Engine > Instance groups.

Click Create Instance group.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	us-central1-mig
Location	Multiple zones
Region	us-central1
Instance template	mywebserver-template
Under Autoscaling metrics, click on the edit pencil icon.

Under Metric type, select HTTP load balancing utilization.

Enter Target HTTP load balancing utilization to 80.

Click Done.

Set Cool down period to 60 seconds.

Enter Minimum number of instances 1 and Maximum number of instances 2.

Managed instance groups offer autoscaling capabilities that allow you to automatically add or remove instances from a managed instance group based on increases or decreases in load. Autoscaling helps your applications gracefully handle increases in traffic and reduces cost when the need for resources is lower. You just define the autoscaling policy, and the autoscaler performs automatic scaling based on the measured load.

For Health check, select Create a health check.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (select option as specified)
Name	http-health-check
Protocol	TCP
Port	80
Managed instance group health checks proactively signal to delete and recreate instances that become unhealthy.

Click Save and continue.

For Initial delay, type 60. This is how long the Instance Group waits after initializing the boot-up of a VM before it tries a health check. You don't want to wait 5 minutes for this during the lab, so you set it to 1 minute.

Click Create.

Click OK.

NOTE: If a warning window will appear stating that There is no backend service attached to the instance group. Ignore this; you will configure the load balancer with a backend service in the next section of the lab.
Repeat the same procedure for europe-west1-mig in europe-west1:

Click Create Instance group.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	europe-west1-mig
Location	Multiple zones
Region	europe-west1
Instance template	mywebserver-template
Autoscaling metrics > Metric Type	HTTP load balancing utilization
Target HTTP load balancing utilization	80
Minimum number of instances	1
Maximum number of instances	2
Cool down period	60
For Health check, select http-health-check (TCP).

For Initial delay, type 60.

Click Create.

Click OK in the dialog window.

Click Check my progress to verify the objective.
Configure an instance template and create instance groups

Verify the backends
Verify that VM instances are being created in both regions.

On the Navigation menu, click Compute Engine > VM instances. Notice the instances that start with us-central1-mig and europe-west1-mig. These instances are part of the managed instance groups.

Task 5. Configure the HTTP load balancer
Configure the HTTP load balancer to balance traffic between the two backends (us-central1-mig in us-central1 and europe-west1-mig in europe-west1) as illustrated in the network diagram:

network_diagram.png

Start the configuration
On the Navigation menu, click Network Services > Load balancing.

Click Create load balancer.

Under HTTP(S) Load Balancing, click Start configuration.

Select From Internet to my VMs, then click Continue.

For Name, type http-lb.

Configure the backend
Backend services direct incoming traffic to one or more attached backends. Each backend is composed of an instance group and additional serving capacity metadata.

Click Backend configuration.

For Backend services & backend buckets, click Create a backend service.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (select option as specified)
Name	http-backend
Backend type	Instance group
Instance group	us-central1-mig
Port numbers	80
Balancing mode	Rate
Maximum RPS	50
Capacity	100
This configuration means that the load balancer attempts to keep each instance of us-central1-mig at or below 50 requests per second (RPS).

Click Done.

Click Add backend.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (select option as specified)
Instance group	europe-west1-mig
Port numbers	80
Balancing mode	Utilization
Maximum backend utilization	80
Capacity	100
This configuration means that the load balancer attempts to keep each instance of europe-west1-mig at or below 80% CPU utilization.

Click Done.

For Health Check, select http-health-check (TCP).

Check the Enable logging checkbox.

Specify Sample rate as 1.

Click Create.

Configure the frontend
The host and path rules determine how your traffic will be directed. For example, you could direct video traffic to one backend and direct static traffic to another backend. However, you are not configuring the host and path rules in this lab.

Click Frontend configuration.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Protocol	HTTP
IP version	IPv4
IP address	Ephemeral
Port	80
Click Done.

Click Add Frontend IP and port.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Protocol	HTTP
IP version	IPv6
IP address	Ephemeral
Port	80
Click Done.

HTTP(S) load balancing supports both IPv4 and IPv6 addresses for client traffic. Client IPv6 requests are terminated at the global load balancing layer and then proxied over IPv4 to your backends.

Review and create the HTTP load balancer
Click Review and finalize.
Review the Backend services and Frontend.
Click Create. Wait for the load balancer to be created.
Click on the name of the load balancer (http-lb).
Note the IPv4 and IPv6 addresses of the load balancer for the next task. They will be referred to as [LB_IP_v4] and [LB_IP_v6], respectively.
The IPv6 address is the one in hexadecimal format.

Click Check my progress to verify the objective.
Configure the HTTP load balancer

Task 6. Stress test the HTTP load balancer
Now that you have created the HTTP load balancer for your backends, it is time to verify that traffic is forwarded to the backend service.


The HTTP load balancer should forward traffic to the region that is closest to you.

True

False

Access the HTTP load balancer
Open a new tab in your browser and navigate to http://[LB_IP_v4]. Make sure to replace [LB_IP_v4] with the IPv4 address of the load balancer.
Accessing the HTTP load balancer might take a couple of minutes. In the meantime, you might get a 404 or 502 error. Keep trying until you see the page of one of the backends.

Stress test the HTTP load balancer
Create a new VM to simulate a load on the HTTP load balancer. Then determine whether traffic is balanced across both backends when the load is high.

In the Cloud Console, on the Navigation menu (Navigation menu), click Compute Engine > VM instances.

Click Create instance.

Specify the following, and leave the remaining settings as their defaults:

Property	Value (type value or select option as specified)
Name	stress-test
Region	us-west1
Zone	us-west1-c
Series	N1
Machine type	f1-micro (1 vCPU)
Because us-west1 is closer to us-central1 than to europe-west1, traffic should be forwarded only to us-central1-mig (unless the load is too high).

For Boot Disk, click Change.

Click Custom images.

For Image, select mywebserver.

Click Select.

Click Create. Wait for the stress-test instance to be created.

For stress-test, click SSH to launch a terminal and connect.

To create an environment variable for your load balancer IP address, run the following command:

export LB_IP=<Enter your [LB_IP_v4] here>
Copied!
Verify it with echo:

echo $LB_IP
Copied!
To place a load on the load balancer, run the following command:

ab -n 500000 -c 1000 http://$LB_IP/
Copied!
Click Check my progress to verify the objective.
Stress test the HTTP load balancer

In the Cloud Console, on the Navigation menu (Navigation menu), click Network Services > Load balancing.
Click Backends.
Click http-backend.
Monitor the Frontend Location (Total inbound traffic) between North America and the two backends for a couple of minutes.
At first, traffic should just be directed to us-central1-mig, but as the RPS increases, traffic is also directed to europe-west1-mig. This demonstrates that by default traffic is forwarded to the closest backend, but if the load is very high, traffic can be distributed across the backends.

In the Cloud Console, on the Navigation menu (Navigation menu), click Compute Engine > Instance groups.
Click on us-central1-mig to open the instance group page.
Click Observability to monitor the number of instances and LB capacity.
Repeat the same for the europe-west1-mig instance group.
Depending on the load, you might see the backends scale to accommodate the load.

Task 7. Review
In this lab, you configured an HTTP load balancer with backends in us-central1 and europe-west1. Then you stress-tested the load balancer with a VM to demonstrate global load balancing and autoscaling.

End your lab