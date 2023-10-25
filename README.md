# Failover
ProjectProperty

# jan/02/1970 00:28:09 by RouterOS 6.48.6
# software id = TNSZ-ZN0D
#
# model = RB750Gr3
# serial number = HD608D303HE
/interface ethernet
set [ find default-name=ether1 ] comment=ISP1 name="ether1-ISP 1(UTAMA)"
set [ find default-name=ether2 ] comment=ISP2 name="ether2-ISP(BACKUP)"
set [ find default-name=ether4 ] comment=CLIENT
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip hotspot profile
set [ find default=yes ] html-directory=hotspot
/ip pool
add name=dhcp_pool1 ranges=192.12.0.2-192.12.0.254
/ip dhcp-server
add address-pool=dhcp_pool1 disabled=no interface=ether4 name=dhcp1
/queue simple
add max-limit=100M/100M name="Total Bandwith" target=192.12.0.0/24
/ip neighbor discovery-settings
set discover-interface-list=!dynamic
/ip address
add address=192.12.0.1/24 comment=Client interface=ether4 network=192.12.0.0
/ip dhcp-client
add add-default-route=no comment=ISP1 disabled=no interface=\
    "ether1-ISP 1(UTAMA)" use-peer-dns=no use-peer-ntp=no
add add-default-route=no comment=ISP2 disabled=no interface=\
    "ether2-ISP(BACKUP)" use-peer-dns=no use-peer-ntp=no
/ip dhcp-server network
add address=192.12.0.0/24 gateway=192.12.0.1
/ip dns
set allow-remote-requests=yes servers=8.8.8.8,8.8.4.4
/ip firewall nat
add action=masquerade chain=srcnat out-interface="ether1-ISP 1(UTAMA)"
add action=masquerade chain=srcnat out-interface="ether2-ISP(BACKUP)"
/ip route
add check-gateway=ping comment="ISP 1 (UTAMA)" distance=1 gateway=\
    192.168.100.1
add check-gateway=ping comment="ISP 2 (BACKUP)" distance=2 gateway=\
    192.168.200.1
add comment="Cek ISP 1" distance=1 dst-address=1.1.1.1/32 gateway=\
    192.168.100.1
add comment="Cek ISP 2" distance=1 dst-address=1.1.2.2/32 gateway=\
    192.168.200.1
/system identity
set name=RouterOS
/tool netwatch
add comment="Failover ISP1" disabled=yes down-script=\
    "/ip route disable [find comment=\"ISP 1 (UTAMA)\"]" host=1.1.1.1 \
    interval=5s up-script="/ip route enable [find comment=\"ISP 1 (UTAMA)\"]"
add disabled=yes down-script=\
    "/ip route disable [find comment=\"ISP 2 (BACKUP)\"]" host=1.1.2.2 \
    up-script="/ip route enable [find comment=\"ISP 2 (BACKUP)\"]"
