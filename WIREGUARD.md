#### WireGuard ####
This is an explaination of different WireGuard concepts on RouterOS

See examples
  * `roles/routeros/wg_push.yaml`
  * `vars/wg88.yaml`



### Hub High-Avability Setup ###
The purpose is the create a HA setup with a floating IP-address between the hub-nodes
The signaling works over a routed VXLAN interface in this example, but a regular bridge och VLAN can be used as well.

    /interface vxlan
    add name=vxlan10 vni=10 mtu=1500 max-fdb-size=16
    
    /interface vxlan vteps
    add interface=vxlan10 remote-ip=172.30.7.2 comment="other-hub"
    
    /interface vrrp
    add interface=vxlan10 name=vxlan10-vrrp \
      on-backup="/interface/wireguard disable [find]" on-master="/interface/wireguard enable [find]"
    
    /ip address
    add address=10.0.0.1/24 interface=wg99 
    add address=172.30.255.2/29 interface=vxlan10 
    add address=172.30.255.1/32 interface=vxlan10-vrrp 
    
    /interface wireguard
    add listen-port=51899 mtu=1500 name=wg99 comment="example service 99"



### Hubs setup (on each hub, per spoke) ###

    /interface wireguard peers
    add name=wg99spoke-XXXX allowed-address=10.0.0.42/32,SPOKE-NETWORK/24 \
      endpoint-address=SPOKE-ENDPOINT endpoint-port=51899 interface=wg99 name=wg99-SPOKE public-key=XXXXXXXX
    
    /ip route
    add comment=wg99-spoke-XXXX dst-address=SPOKE-NETWORK/24 gateway=10.0.0.42 check-gateway=ping



### Spoke setup ###

    /interface wireguard
    add listen-port=51899 mtu=1500 name=wg99 comment="example service 99"
    
    /interface wireguard peers
    add allowed-address==10.0.0.1/32,198.51.100.0/24 \
      endpoint-address=172.30.255.1 endpoint-port=51899 interface=wg99 name=wg99-hub public-key=XXXXXXXXXX
    
    /ip address
    add address=10.0.0.42/24 interface=wg99
    
    /ip route
    add dst-address=198.51.100.0/24 gateway=wg99


