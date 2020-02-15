---
layout:     post
title:      "Manually binding Neutron port"
date:       2020-02-15 16:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [programming, linux, openstack, networking]
comments:   true

---
In an OpenStack cluster, the nova service is responsible for binding the port to the host and attaching it to the virtual machine. We will only
look at openvswitch driver in this post.

<!--more-->

This post is meant for a very niche section of people and I won't be bothering with covering the background required to understand this
post as it will take an entire post series to do so.

First, we need to create a neutron port.

{% highlight bash %}
$ openstack port create --network gre-net --host server1 --device-owner server1.compute 
+--------------------+-----------------------------------------------------------------+
|Field               |Value                                                            |
+--------------------+-----------------------------------------------------------------+
|admin_state_up      |True                                                             |
|binding:capabilities|{"port_filter": false}                                           |
|binding:vif_type    |ovs                                                              |
|device_id           |                                                                 |
|device_owner        |                                                                 |
|fixed_ips           |{"subnet_id": "15a09f6c-87a5-4d14-b2cf-03d97cd4b456", "ip_address": "192.168.2.2"} |
|id                  |baf13412-2641-4183-9533-de8f5b91444c                             |
|mac_address         |fa:16:3e:f6:ec:c7                                                |
|name                |                                                                 |
|network_id          |2d627131-c841-4e3a-ace6-f2dd75d                                  |
|status              |DOWN                                                             |
|tenant_id           |3671f46ec35e4bbca6ef92ab7975e463                                 |
+--------------------+-----------------------------------------------------------------+
{% endhighlight %}

Next, we need to create the virtual interfaces for this port. Nova service creates a veth pair with one end attached to the integration
bridge and other end attached to the VM. But for our case, we will attach the other end on the host itself. We need to follow the
naming convention of neutron for the veth peer that will be attached to the integration bridge. Otherwise, the neutron agent will not
recognize this port and perform its side of the job.


We will create a veth pair with one end with the name "tap<first 11 digits of neutron port id>". In this case, it would be tapbaf13412-26.
There is no restriction on the naming of the other end(we will call it host_port for the rest in the post). We need to assign IP and mac
address of the neutron port to the host_port. Having the correct IP and mac address is vital as this is the interface
that will be responsible for creating the packets that go into the integration bridge. And without the correct address the neutron agent will
drop these packets. 

{% highlight bash %}
$ ip link add dev host_port type veth peer name tapbaf13412-26
$ ip link set dev host_port address fa:16:3e:f6:ec:c7
$ ip link set host_port up
$ ip link set tapbaf13412-26 up
$ ip addr add 192.168.2.2/24 dev host_port
{% endhighlight %}

Next, we need to add the other side to the neutron integration bridge (default name br-int).

{% highlight bash %}
$ ovs-vsctl add-port br-int tapbaf13412-26
{% endhighlight %} 

After this, the neutron agent running on the host will recognize this interface due to its ID and mark the corresponding neutron port
ACTIVE. This completes the binding process.

I will further cover a bit of the background of the problem that I faced and how manually binding a port helped me to resolve the issue.

# Why is it needed 

I had an OpenStack cluster that was using GRE tunnels to allow communication between VMs. If you create a neutron network over a GRE 
tunnel, your VMs communicate through the tunnel. But you cannot access this network from the compute host.

The problem is slightly complicated then it initially looks. The below figure shows the ovs configuration on the compute host.

<img class="center-image" src="{{ site.baseurl }}/assets/images/normal_vm.png" style="width:60%"/>

The first solution that comes to mind after looking at the diagram is to add an ovs port on br-tun, which would be accessible from the
host. But the problem with this approach are the vlan IDs. The neutron agent adds rules to the br-tun which only allow the packets with
specific vlan IDs into the GRE tunnels. And these vlan IDs correspond to the ones assigned by the neutron agent on the ports attached
to br-int. These vlan IDs keeps changing across restarts of neutron agent. This also means that any solution that involves having
a port on br-int won't work as we need to assign tag to it dynamically. I wanted to avoid having a solution that includes a
continuously running component that updates the changes done by neutron agent. 

The solution to the problem was working with neutron agent and not copying it. That's when the idea of binding the neutron port to the host came to my
mind. Since it is a neutron port, the neutron agent will take care of assinging it the correct tags and ensuring the connectivity. The
below figure shows the configuration that I used.

<img class="center-image" src="{{ site.baseurl }}/assets/images/host_port.png" style="width:60%"/>
