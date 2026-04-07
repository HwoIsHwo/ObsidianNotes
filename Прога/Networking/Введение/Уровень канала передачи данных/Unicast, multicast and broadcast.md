![[Pasted image 20230916133548.png]]

A unicast transmission is always meant for just one receiving address.

At the ethernet level this is done by looking at a special bit in the destination MAC address. If the least significant bit in the first octet of a destination address is set to zero, it means that ethernet frame is intended for only the destination address.

If the least significant bit in the first octet of a destination address is set to one, it means you're dealing with a multicast frame.

![[Pasted image 20230916133926.png]]

The third type of ethernet transmission is known as broadcast. In ethernet, broadcast is sent to every single device on a LAN. This is accomplished by using a special destination known as a broadcast address. The ethernet broadcast address is all F's. Ethernet broadcasts are used so that devices can learn more about each other.

![[Pasted image 20230916134016.png]]

![[Pasted image 20230916134141.png]]

Теги: #Прога 
Ссылки:
[[Ethernet and MAC addresses]]
[[Маршрутизаторы]]