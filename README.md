# RouterOS Ansible Provider tools #

This is a set of tasks and data strucutres to manage a large set of Layer 2 or Layer 3/VRF-attached tunnels. This should be seen more as a proof-of-concept or inspiration.


### Current focus ###

  - Orchestrate a large set of L2 tunnels, bridges and VRFs, thus creating virtual services across a network.
  - Orchestrate a large set of L3 tunnels, bridges and VRFs, thus creating virtual services across a network.
  - Manage VLAN interfaces
  - Manage directly routed networks/subnets
  - Manage directly routed networks/subnets via OSPF or BGP
  - General and common settings
  - RouterOS 6 and 7 compability (VXLAN only supported in 7.x)


### Future plans ###

  - Users and security
  - Switching, VLANs and Bridges
  - Additional Tunnel settings


### Types ###

  - **eoip** - Plain EoIP tunnel
  - **eoip_ab** - Active/Backup setup of a EoIP-tunnel. Bound to a floating IP-address (like VRRP) between the hub routers.
  - **vxlan** - Enables Layer 2 Hub to Spoke or Spoke-to-Spoke L2VPN with over IP-transport


#### EoIP ####

EoIP (Ethernet-over-IP) is a Mikrotik RouterOS properatary GRE-variant to transport Ethernet-frames over IP-transport. It has ability for handling fragmentation and other stuff.

  - **l2t** - Simple Layer 2 tunnel
  - **l2br** - Bridged Layer 2 tunnel (bridged at hubs)
  - **l3t** - Simple Layer 3 tunnel


#### EoIP Tunnel options ####

Tunnels are defined in `vars/routeros_tunnel.yaml`

    service_id                       Service ID, Bridge ID or VRF ID
    service_type                     Type of tunnel
    tun_id                           ID of this tunnel within Service ID
    type                             Type of tunnel
    hub_node                         Single hub hostname
    hub_primary                      Primary hub hostname
    hub_backup                       Secondary hub hostname (if applicable)
    hub_source                       Floating IP-address between the primary/secondary hubs
    hub_address                      Hub IP-adress within the VRF, within the tunnel (for example customer network)
    hub_vlan                         For Bridged L2 service, VLAN on common bridge/trunk between hubs
    end_node                         Endpoint hostname
    end_intf                         Endpoint interface or vlan-interface
    comment                          Optional comment
    state                            State of tunnel: absent/present/shutdown


#### EoIP Hub configuration ####

The hubs should share a floating IP-adress of some kind, for example with VRRP. It could also be a loopback-adress handled by for example OSPF. In addition to this you should configure `on-master`/`on-backup` actions that enable or disables Active/Backup tunnels according to the active hub. You could use the VRRP adress (172.30.255.1 as bellow) as key for setting state of the tunnels.

    /interface vrrp
    interface=bridge name=vrrp-cluster \
    on-backup="/interface/eoip/disable [find local-address=172.30.255.1]" \
    on-master="/interface/eoip/enable [find local-address=172.30.255.1]"

This is primarly to make sure that the tunnel subnet is removed from the routing table (most likley a VRF) when tunnel is not longer runnning. If you plan to use Bridged Layer2 services, you should provide a direct link between the hubs. This link can also house the VRRP-adress on a VLAN-interface.


#### EoIP considerations ####

Mikrotik EoIP tunnels uses a 16-bit (1-65,535) Tunnel ID. Because of this you will need to decide early if you are going to settle two digit or three digit services (1-64 or 1-649). These ansible tasks concatenates the service ID with the tunnel ID to form a tunnel ID, but it can never go above 65,535. So:

  * 2 digit services will result in more possible tunnels (1-999 per service)
  * 3 digit services will result in fewer possible tunnels (1-99 per service)




### VXLAN ###

  - **bridge** - Meshed bridge service at hub nodes (A and B)
  - **etree** - Hub-Spoke E-Tree like L2 service
  - **eline** - Spoke-Spoke E-Line like L2 service


#### VXLAN Tunnel options ####

See examples under `vars/`

    vni                              VXLAN Virtual Network Identifier number (1-16777216)
    bridge                           Bridge name (default: "vxbridge"). This allows for multiple "bridge domains".
    bridge_vlan                      VLAN used in the bridge above (1-4094)
    spoke_node                       Endpoint hostname
    spoke_interface                  Endpoint interface to attach to service
    node_a                           Node A hostname
    node_a_interface                 Node A interface to attach to service
    node_b                           Node B hostname
    node_b_interface                 Node B interface to attach to service
    comment                          Optional comment
    state                            State of tunnel: absent/present/shutdown


#### VXLAN Hub configuration ####

The Ansible-tasks will create the common bridge if missing, but will not setup any additional trunks. The most common use is to setup a trunk-port to a router or a firewall that will act as the Layer 3 gateway for the Layer 2 segment. Another solution is to add the bridge itself as tagged member, and create VLAN-interfaces that resides in different VRFs as required.

Example:

    /interface bridge vlan
    add bridge=vxbridge tagged=ether4-to-firewall vlan-ids=1-4094
    add bridge=vxbridge untagged=vxlan101 vlan-ids=101
    add bridge=vxbridge untagged=vxlan102 vlan-ids=102
    add bridge=vxbridge untagged=vxlan103 vlan-ids=103


#### VXLAN considerations ####

VXLAN is mainly designed for *in-datacenter* and/or *between-datacenters* usage on private lines or leased lines/capacity. It's not designed for *over-public-internet* usage. There are no security in place and the whole transport-path should be controlled by yourself. Fragmentation is neither handled so it requires you to have needed MTU headroom.

It is however fully possible to setup IPsec encryption between nodes as with all IP-traffic.

VXLAN requires both parties to have a VTEP relation to work. For that reason, by design, Spokes in a E-tree setup will not be able to communicate, hence the E-tree name.


#### WireGuard ####


  - spoke: skapa vrf
  - spoke: skapa wg-interface, spara undan pubkey
  - spoke: skapa peers med endpoints och allowed-addresses
  - spoke: skapa bgp-instans
  - hubs: skapa spoke med endpoint och allowed-addresses



## Spoke setup
/interface list
add name=VRF30 

/ip vrf
add interfaces=vrf30 interfaces=VRF30

/interface wireguard
add listen-port=13030 mtu=1400 name=wg30

/interface list member
add list=VRF30 interface=wg30
add list=VRF30 interface=ether7

/ip address
add address=SPOKE-TUNNEL-IP/32 interface=wg30

/interface wireguard peers
add allowed-address=HUB-TUNNEL-IP1/32,HUB-NETWORK1 client-address=SPOKE-TUNNEL-IP/32 endpoint-address=HUB-IP1 endpoint-port=13030 interface=wg30 name=wg30-hub1 public-key=""

add allowed-address=HUB-TUNNEL-IP2/32,HUB-NETWORK2 client-address=SPOKE-TUNNEL-IP/32 endpoint-address=HUB-IP2 endpoint-port=13030 interface=wg30 name=wg30-hub2 public-key=""

/routing bgp connection
add address-families=ip as=65000 connect=yes disabled=no listen=no local.address=SPOKE-TUNNEL-IP .role=ibgp name=wg30-hub1 nexthop-choice=force-self output.redistribute=connected remote.address=HUB-TUNNEL-IP1 .as=65000 routing-table=VRF30 vrf=VRF30

add address-families=ip as=65000 connect=yes disabled=no listen=no local.address=SPOKE-TUNNEL-IP .role=ibgp name=wg30-hub2 nexthop-choice=force-self output.redistribute=connected remote.address=HUB-TUNNEL-IP2 .as=65000 routing-table=VRF30 vrf=VRF30




