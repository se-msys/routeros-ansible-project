# routeros-ansible-project
Manages services on a set of RouterOS devices. This should be seen more as a proof-of-concept or inspiration.

### Current focus ###

  - Orchestrate a large set of L2 tunnels, bridges and VRFs, thus creating virtual services across a network
  - Manage VLAN interfaces
  - Manage directly routed networks/subnets via OSPF or BGP
  - General and common settings
  - Both RouterOS 6 and 7 compability

### Future goals ###

  - Users and security
  - Switches, VLANs and bridges
  - Additional tunnel settings


### Examples ###

Detect version, and deploy provider hub services

    ansible-playbook pb-routeros.yml -i hosts -t detect,provider -l hub1,hub2

Detect version, and deploy tunnels for EoIP and VPLS

    ansible-playbook pb-routeros.yml -i hosts -t detect,vpls,eoip


