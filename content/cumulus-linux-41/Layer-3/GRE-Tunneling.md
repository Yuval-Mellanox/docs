---
title: GRE Tunneling
author: NVIDIA
weight: 880
toc: 3
---
{{<notice warning>}}

GRE Tunneling is an {{<kb_link url="knowledge-base/Support/Support-Offerings/Early-Access-Features-Defined/">}}.

{{</notice>}}

Generic Routing Encapsulation (GRE) is a tunneling protocol that encapsulates network layer protocols inside virtual point-to-point links over an Internet Protocol network. The two endpoints are identified by the tunnel source and tunnel destination addresses at each endpoint.

GRE packets travel directly between the two endpoints through a virtual tunnel. As a packet comes across other routers, there is no interaction with its payload; the routers only parse the outer IP packet. When the packet reaches the endpoint of the GRE tunnel, the outer packet is de-encapsulated, the payload is parsed, then forwarded to its ultimate destination.

GRE uses multiple protocols over a single-protocol backbone and is less demanding than some of the alternative solutions, such as VPN. You can use GRE to transport protocols that the underlying network does not support, work around networks with limited hops, connect non-contiguous subnets, and allow VPNs across wide area networks.

{{%notice note%}}

- GRE tunneling is supported for switches with {{<exlink url="https://cumulusnetworks.com/products/hardware-compatibility-list/?asic%5B0%5D=Mellanox%20Spectrum&asic%5B1%5D=Mellanox%20Spectrum_A1" text="Spectrum ASICs">}} only.
- Only static routes are supported as a destination for the tunnel interface.
- IPv6 endpoints are not supported.

{{%/notice%}}

The following example shows two sites that use IPv4 addresses. Using GRE tunneling, the two end points can encapsulate an IPv4 or IPv6 payload inside an IPv4 packet. The packet is routed based on the destination in the outer IPv4 header.

{{< img src = "/images/cumulus-linux/gre-tunnel-example.png" >}}

## Configure GRE Tunneling

To configure GRE tunneling, you create a GRE tunnel interface with routes for tunneling on both endpoints as follows:

1. Create a tunnel interface by specifying an interface name, the tunnel mode as `gre`, the source (local) and destination (remote) underlay IP address, and the `ttl` (optional).
2. Bring the GRE tunnel interface up.
3. Assign an IP address to the tunnel interface.
4. Add route entries to encapsulate the packets using the tunnel interface.

The following configuration example shows the commands used to set up a bidirectional GRE tunnel between two endpoints: `Tunnel-R1` and `Tunnel-R2`. The local tunnel endpoint for `Tunnel-R1` is 10.0.0.9 and the remote endpoint is 10.0.0.2. The local tunnel endpoint for `Tunnel-R2` is 10.0.0.2 and the remote endpoint is 10.0.0.9.

{{< img src = "/images/cumulus-linux/gre-tunnel-config.png" >}}

**Tunnel-R1 commands:**

```
cumulus@switch:~$ sudo ip tunnel add Tunnel-R2 mode gre remote 10.0.0.2 local 10.0.0.9 ttl 255
cumulus@switch:~$ sudo ip link set Tunnel-R2 up
cumulus@switch:~$ sudo ip addr add 10.0.100.1 dev Tunnel-R2
cumulus@switch:~$ sudo ip route add 10.0.100.0/24 dev Tunnel-R2
```

**Tunnel-R2 commands:**

```
cumulus@switch:~$ sudo ip Tunnel add Tunnel-R1 mode gre remote 10.0.0.9 local 10.0.0.2 ttl 255
cumulus@switch:~$ sudo ip link set Tunnel-R1 up
cumulus@switch:~$ sudo ip addr add 10.0.200.1 dev Tunnel-R1
cumulus@switch:~$ sudo ip route add 10.0.200.0/24 dev Tunnel-R1
```

To apply the GRE tunnel configuration automatically at reboot, instead of running the commands from the command line (as above), you can add the following commands directly in the `/etc/network/interfaces` file.

```
cumulus@switch:~$ sudo nano /etc/network/interfaces
...
# Tunnel-R1 configuration
auto swp1 #underlay interface for tunnel
iface swp1
    link-speed 10000
    link-duplex full
    link-autoneg off
    address 10.0.0.9/24

auto Tunnel-R2
iface Tunnel-R2
    tunnel-mode gre
    tunnel-endpoint 10.0.0.2
    tunnel-local 10.0.0.9
    tunnel-ttl 255
    address 10.0.100.1
    up ip route add 10.0.100.0/24 dev Tunnel-R2

# Tunnel-R2 configuration
auto swp1 #underlay interface for tunnel
iface swp1
    link-speed 10000
    link-duplex full
    link-autoneg off
    address 10.0.0.2/24

auto Tunnel-R1
iface Tunnel-R1
    tunnel-mode gre
    tunnel-endpoint 10.0.0.9
    tunnel-local 10.0.0.2
    tunnel-ttl 255
    address 10.0.200.1
    up ip route add 10.0.200.0/24 dev Tunnel-R1
```

## Verify GRE Tunnel Settings

To check GRE tunnel settings, run the `ip tunnel show` command or the `ifquery --check` command. For example:

```
cumulus@switch:~$ ip tunnel show
gre0: gre/ip remote any local any ttl inherit nopmtudisc
Tunnel-R1: gre/ip remote 10.0.0.2 local 10.0.0.9 ttl 255
```

```
cumulus@switch:~$ ifquery --check Tunnel-R1
auto Tunnel-R1
iface Tunnel-R1                                                 [pass]
        up ip route add 10.0.200.0/24 dev Tunnel-R1                 []
        tunnel-ttl 255                                          [pass]
        tunnel-endpoint 10.0.0.9                                [pass]
        tunnel-local 10.0.0.2                                   [pass]
        tunnel-mode gre                                         [pass]
        address 10.0.200.1/32                                   [pass]
```

## Delete a GRE Tunnel Interface

To delete a GRE tunnel, remove the tunnel interface, and remove the routes configured with the tunnel interface, run the `ip tunnel del` command. For example:

```
cumulus@switch:~$ sudo ip tunnel del Tunnel-R2 mode gre remote 10.0.0.2 local 10.0.0.9 ttl 255
```

{{%notice note%}}

You can delete a GRE tunnel directly from the `/etc/network/interfaces` file instead of using the `ip tunnel del` command. Make sure you run the `ifreload - a` command after you update the interfaces file.

This action is disruptive as the tunnel is removed, then recreated with the new settings.

{{%/notice%}}

## Change GRE Tunnel Settings

Use the `ip tunnel change` command to make changes to the GRE tunnel settings. The following example changes the remote underlay IP address from the original setting to 11.0.0.4:

```
cumulus@switch:~$ sudo ip tunnel change Tunnel-R2 mode gre local 10.0.0.2 remote 10.0.0.4
```

{{%notice note%}}

You can make changes to GRE tunnel settings directly in the `/etc/network/interfaces` file instead of using the `ip tunnel change` command. Make sure you run the `ifreload - a` command after you update the interfaces file.

{{%/notice%}}
