# RouterOS Ansible Provider tools #

This is a set of tasks and data strucutres to manage a large set of Layer 2 or Layer 3/VRF-attached tunnels. This should be seen more as a proof-of-concept or inspiration.


### Current focus ###

  - L3VPN with WireGuard Hub(s) to Spoke scenarios
  - L2VPN with VXLAN with E-tree and E-line scenarios - with additional IPsec
  - L3VPN with EoIP - with our without dynamic routing

### Future plans ###

  - Users and security
  - Switching, VLANs and Bridges
  - Additional Tunnel settings

#### EoIP with inventory ####

  - **eoip** - Plain EoIP tunnel
  - **eoip_ab** - Active/Backup setup of a EoIP-tunnel. Bound to a floating IP-address (like VRRP) between the hub routers.

EoIP (Ethernet-over-IP) is a Mikrotik RouterOS properatary GRE-variant to transport Ethernet-frames over IP-transport. It has ability for handling fragmentation and other stuff.

  - **l2t** - Simple Layer 2 tunnel
  - **l2br** - Bridged Layer 2 tunnel (bridged at hubs)
  - **l3t** - Simple Layer 3 tunnel


#### EoIP inventory options ####

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
  - **etree** - Hubs-Spoke E-Tree like L2VPN service
  - **eline** - Spoke-Spoke E-Line like L2VPN service


#### VXLAN Tunnel options ####

See examples under `vars/routeros_vxlan_*.yaml`

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


#### WireGuard tunnels ####

To be documented

See `roles/routeros/tasks/wg_push.yaml` and `vars/wg99.yaml` for example use

Can be setup in a high-avability fashion with the help of VXLAN and VRRP


    /interface vxlan
    add name=vxlan10 vni=10 mtu=1500 max-fdb-size=16
    
    /interface vxlan vteps
    add interface=vxlan10 remote-ip=172.30.7.2 comment="other-hub"
    
    /interface vrrp
    add interface=vxlan10 name=vxlan10-vrrp \
      on-backup="/interface/wireguard disable [find]" on-master="/interface/wireguard enable [find]"
    
    /ip address
    add address=172.30.255.2/29 interface=vxlan10 
    add address=172.30.255.1/32 interface=vxlan10-vrrp 
    


#### EoIP tunnels ####

To be documented

See `roles/routeros/tasks/eoip_push.yaml` and `vars/eoip55.yaml` for example use






