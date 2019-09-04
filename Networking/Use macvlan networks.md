# Use macvlan networks
- Some applications, especially legacy applications or applications which monitor network traffic, expect to be directly connected to the physical network.
- In this type of situation, you can use the macvlan network driver to assign a MAC address to each container’s virtual network interface, making it appear to be a physical network interface directly connected to the physical network.
- In this case, you need to designate a physical interface on your Docker host to use for the macvlan, as well as the subnet and gateway of the macvlan. You can even isolate your macvlan networks using different physical network interfaces.
# Create a macvlan network
## Bridge mode
```
docker network create -d macvlan \
  --subnet=172.16.86.0/24 \
  --gateway=172.16.86.1 \
  -o parent=eth0 pub_net
```
### 802.1q trunk bridge mode
```
docker network create -d macvlan \
    --subnet=192.168.50.0/24 \
    --gateway=192.168.50.1 \
    -o parent=eth0.50 macvlan50
```
# Use an ipvlan instead of macvlan
In the above example, you are still using a L3 bridge. You can use ipvlan instead, and get an L2 bridge. Specify -o ipvlan_mode=l2.
```
docker network create -d ipvlan \
    --subnet=192.168.210.0/24 \
    --subnet=192.168.212.0/24 \
    --gateway=192.168.210.254 \
    --gateway=192.168.212.254 \
     -o ipvlan_mode=l2 ipvlan210
```
