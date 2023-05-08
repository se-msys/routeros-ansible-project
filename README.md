# RouterOS Ansible Provider tools

This is a set of tasks and data strucutres to manage a large set of Layer2 or Layer3/VRF-attached tunnels. This should be seen more as a proof-of-concept or inspiration.

### Current focus ###

  - Orchestrate a large set of L2 tunnels, bridges and VRFs, thus creating virtual services across a network.
  - Orchestrate a large set of L3 tunnels, bridges and VRFs, thus creating virtual services across a network.
  - Manage VLAN interfaces
  - Manage directly routed networks/subnets
  - Manage directly routed networks/subnets via OSPF or BGP
  - General and common settings
  - RouterOS 6 and 7 compability


### Future plans ###

  - Users and security
  - Switches, VLANs and bridges
  - Additional tunnel settings


### Service types ###

  - **l2t** - Simple Layer 2 tunnel
  - **l2br** - Bridged Layer 2 tunnel (bridged at hubs)
  - **l3t** - Simple Layer 3 tunnel


### Tunnel types ###

  - **eoip** - Plain EoIP tunnel
  - **eoip_ab** - Active/Backup setup of a EoIP-tunnel. Bound to a floating IP-address (like VRRP) between the hub routers.
  - **vpls** - Plain VPLS tunnel
  - **vpls_ab** - Active/Backup setup of a VPLS-tunnel. Bound to a floating IP-address (like VRRP) between the hub routers.


### Tunnel options ###

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


### Hub configuration ###

The hubs should share a floating IP-adress of some kind, for example with VRRP. It could also be a loopback-adress handled by for example OSPF. In addition to this you should configure `on-master`/`on-backup` actions that enable or disables Active/Backup tunnels according to the active hub. You could use the VRRP adress (172.30.255.1 as bellow) as key for setting state of the tunnels.

    /interface vrrp
    interface=bridge name=vrrp-cluster \
    on-backup="/interface/eoip/disable [find local-address=172.30.255.1]" \
    on-master="/interface/eoip/enable [find local-address=172.30.255.1]"

This is primarly to make sure that the tunnel subnet is removed from the routing table (most likley a VRF) when tunnel is not longer runnning. If you plan to use Bridged Layer2 services, you should provide a direct link between the hubs. This link can also house the VRRP-adress on a VLAN-interface.


### EoIP (Ethernet-over-IP) considerations ###

Mikrotik EoIP tunnels uses a 16-bit (1-65,535) Tunnel ID. Because of this you will need to decide early if you are going to settle two digit or three digit services (1-64 or 1-649). These ansible tasks concatenates the service ID with the tunnel ID to form a tunnel ID, but it can never go above 65,535. So:

  * 2 digit services will result in more possible tunnels (1-999 per service)
  * 3 digit services will result in fewer possible tunnels (1-99 per service)

VPLS uses a 2byte + 4byte or 4byte + 2byte identifier, so it doesn't suffer from the same restrictions.




