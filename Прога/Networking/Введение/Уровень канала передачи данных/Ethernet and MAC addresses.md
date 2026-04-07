Ethernet - the protocol most widely used to send data across individual links.

Ethernet as a protocol solved the problem of a collision by using a technique known as carrier sense ,ultiple access with collision detection.
(Протокол Ethernet решает эту проблему, используя метод, известный как множественный доступ с контролем несущей и обнаружением коллизий.)

CSMA/CD
Used to determine when the communications channels are clear and when a device is free to transmit data. 

![[Pasted image 20230916132209.png]]

If there's no data currently being transmitted on the network segment, a node will feel free to send data. If it turns out that two or more computers end up trying to send data at the same time, the computers detect this collision and stop sending data. Each device involved with the collision then waits a random interval of time before trying to send data again. This random interval, helps to prevent all the computers involved in the collision from colliding again the next time they try to transmit anything. When a network segment is a collision domain, it means that all devices on that segment receive all communication across the entire segment. This means we need a way to identify which node the transmission was actually meant for. This is where something known as a media access control address or MAC address comes into play.

MAC address
A globally unique identifier attached to an individuel network interface. It's a 48-bit number normally represented by six groupings of two hexadecimal numbers.

Octet 
in computer networking, any number thet can represented by 8 bits.

Organizationally Unique Identifier (OUI)
The first three ictets of a MAC address. 

![[Pasted image 20230916133010.png]]

The last three octets of a MAC address
can be assigned in any way that the manufacturer would like, with the condition that they only assign each possible address once to keep all MAC addresses globally unique. Ethernet uses MAC addresses to ensure that the data it sends has both an address for the machine that sent the transmission, as well as, the one that the transmission was intended for.

Теги: #Прога 
Ссылки:
[[Концентраторы и коммутаторы]]