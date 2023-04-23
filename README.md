# RouterOS Ansible Provider tools

This is a set of tasks and data strucutres to manage a large set of Layer2 or Layer3/VRF-attached tunnels. This should be seen more as a proof-of-concept or inspiration.

### Current focus ###

  - Orcehstrate a large set of L2 tunnels, bridges and VRFs, thus creating virtul services across a network.
  - Orchestrate a large set of L2 tunnels, bridges and VRFs, thus creating virtual services across a network
  - Manage VLAN interfaces
  - Manage directly routed networks/subnets
  - Manage directly routed networks/subnets via OSPF or BGP
  - General and common settings
  - Both RouterOS 6 and 7 compability

### Future goals ###

  - Users and security
  - Switches, VLANs and bridges
  - Additional tunnel settings



### Tunnels types ###

  - "l2t" - Simple Layer 2 tunnel
  - "l2br" - Bridged Layer 2 tunnel
  - "l3t" - Simple Layer 3 tunnel
  - eoip - Plain EoIP tunnel
  - eoip_ap - Active/Passive setup of a EoIP-tunnel. Bound to a floating IP-address (like VRRP) between the hub routers.
  - vpls - Plain VPLS tunnel


### Tunnels options ###

    service_id                       Service ID, Bridge ID, VRF IF
    service_type                     Type of tunnel
    tun_id                           ID of this tunnel within Service ID
    type                             Type of tunnel
    hub_primary                      Primary hub hostname
    hub_backup                       Secondary hub hostname
    hub_source                       Floating IP-address between the primary/secondary hubs
    hub_address                      Hub IP-adress within the VRF, within the tunnel (for example customer network)
    end_node                         Endpoint hostname
    end_intf                         Endpoint interface or vlan-interface
    comment                          Optional comment
    state                            State of tunnel: absent/presen/shutdown


