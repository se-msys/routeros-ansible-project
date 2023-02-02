# routeros-ansible-project
Manages services on a set of RouterOS devices

### Focus and goals ###

  - Orcehstrate a large set of L2 tunnels, bridges and VRFs, thus creating virtul services across a network.
  - Manage VLAN interfaces
  - Manage directly routed networks/subnets
  - General and common settings


### Examples ###

Detect version, and deploy provider hub services

    ansible-playbook pb-routeros.yml -i hosts -t detect,provider -l hub1,hub2

Detect version, and deploy tunnels for EoIP and VPLS

    ansible-playbook pb-routeros.yml -i hosts -t detect,vpls,eoip


