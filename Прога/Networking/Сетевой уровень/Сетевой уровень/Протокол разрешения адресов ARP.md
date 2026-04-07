Протокол разрешения адресов, или ARP. ARP — это протокол, используемый для определения аппаратного адреса узла с определенным IP-адресом.

После того, как IP-дейтаграмма полностью сформирована, ее необходимо инкапсулировать в кадр Ethernet. Это означает, что передающему устройству необходим MAC-адрес назначения для завершения заголовка кадра Ethernet. Практически все сетевые устройства хранят локальную таблицу ARP. Таблица ARP — это просто список IP-адресов, и к каждому из них привязан MAC-адрес.

![[Pasted image 20230917212917.png]]
![[Pasted image 20230917212924.png]]
![[Pasted image 20230917212953.png]]
![[Pasted image 20230917213007.png]]
![[Pasted image 20230917213029.png]]

ARP table entries generally expire after a short amount of time to ensure changes in the network are accounted for.

Теги: #Прога 
Ссылки:
[[IPv4 адреса]]
[[Ethernet and MAC addresses]]
[[Unicast, multicast and broadcast]]
