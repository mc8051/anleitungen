## WLAN Accesspoint einrichten

    apt-get install hostapd isc-dhcp-server

/etc/hostaptd/hostapd.conf

    interface=wlan0
    ssid=gurkengewuerz.de
    hw_mode=g
    channel=6
    macaddr_acl=0
    ignore_broadcast_ssid=0
    auth_algs=1
    wpa=2
    wpa_passphrase=pw_here
    wpa_key_mgmt=WPA-PSK
    wpa_pairwise=TKIP
    rsn_pairwise=CCMP

Config Check

    sudo hostapd /etc/hostapd/hostapd.conf


/etc/wpa_supplicant/wpa_supplicant.conf

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
    update_config=1


/etc/network/interfaces

    auto lo
    iface lo inet loopback

    auto eth0
    allow-hotplug eth0
    iface eth0 inet manual

    allow-hotplug wlan0
    iface wlan0 inet static
        address 192.168.26.1
        netmask 255.255.255.0



/etc/dhcp/dhcpd.conf

    subnet 192.168.26.0 netmask 255.255.255.0 {
        range 192.168.26.5 192.168.26.205;
        option domain-name-servers 8.8.8.8, 8.8.4.4;
        option routers 192.168.26.1;
    }


/etc/rc.local

    sudo service dnsmasq stop
    sudo ifconfig wlan0 192.168.26.1 netmask 255.255.255.0 && sudo /etc/init.d/isc-dhcp-server restart
    sudo iptables -A FORWARD -o eth0 -i wlan0 -m conntrack --ctstate NEW -j ACCEPT
    sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
    sudo iptables -t nat -F POSTROUTING
    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    sudo hostapd /etc/hostapd/hostapd.conf

**_Hinweis:_** es dürfen KEINE zwei oder mehr DNS Server auf einem System laufen, deshalb wird der dnsmasq gestoppt!
