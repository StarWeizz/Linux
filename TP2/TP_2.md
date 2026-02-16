# B1 Linux - TP2

## I. Exploration locale en solo



## II. Exploration locale en duo
- 1. Cablage effectué

- 2. Création du réseau

- 3. Modification d'adresse IP
bien effectuée:

![ll](./images/image%20copy.png)



- 4. Utilisation d'un des deux comme gateway

j'ai decidé de prendre Antonin comme duo pour ce TP, ici ce sera lui qui jouera le role de gateway pendant que moi je joue le role du pc qui n'a plus internet.

je peux bien ping antonin :

![ll](./images/image.png)

et voilà bien un ping 8.8.8.8 qui fonctionne sans carte WiFi :

![ll](./images/image%20copy%202.png)

- 5. Petit chat privé
je suis sur linux donc j'ai deja netcat, et la communication depuis le port 8888 est bien effectuée : 

![ll](./images/mini_chat.png)

- 6. Wireshark


voici ce qu'il se passe dans mon wireshark quand Antonin me ping : 


![ll](./images/image%20copy%203.png)

voici ce qu'il se passe dans mon wireshark quand Antonin netcat : 

![ll](./images/image%20copy%204.png)


- 7. Firewall 