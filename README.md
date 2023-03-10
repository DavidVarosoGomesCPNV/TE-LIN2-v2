# Evaluation du module LIN2 [Docker]
Classe : SI-T1b
Date : 26 janvier 2023 de 13h30 à 15h00 (2 périodes)

## 1. Objectifs

- Connaître les caractéristiques d’un conteneur Linux
- Savoir installer et utiliser Docker
- Maîtriser la création des images Docker et les Dockerfile
- Savoir interagir avec le Docker Hub et les registres privés
- Maîtriser les notions réseaux de Docker (networks, links)
- Maîtriser la gestion des données avec Docker (volumes)
- Savoir utiliser Docker Compose

## 2. Introduction

Lors de cette évaluation, vous allez automatiser la phase de construction et déployer deux conteneurs Docker. Deux `Dockerfile`  et un `docker-compose.yaml` seront remis à la fin de l’épreuve. Un dépôt personnel docker sera créé pour héberger les 2 images précédemment créées. Un second `docker-compose.yaml` sera créé pour déployer l’infrastructure à partir de votre Docker Hub personnel.  Une démonstration du fonctionnement de votre infrastructure déployée à partir de votre dépôt personnel devra être réalisée avant la fin de votre épreuve. 

<aside>
⚠️ Pour réaliser cette épreuve, il vous faudra respecter scrupuleusement les conventions de nommage et de configuration demandées.
</aside>

## 3. Applications

Il vous sera demandé de construire deux conteneurs simples permettant de communiquer entre eux. Les deux applications python (`socket_server.py` et `socket_client.py`) sont fournis et doivent uniquement être déployés, il ne s'agit donc pas de développement, mais de mise en production, orienté système et services.

<aside>
⚠️ Le développeur vous a fourni les applications sans vous spécifier la version de python à utiliser.  A vous de choisir la version de python la plus récente compatible avec les deux applications fournies.
</aside>

Vous allez donc déployer à l’aide de docker une application serveur et une application cliente en python.

1. L’API `serveur` s’exécute en premier et attend une interaction du client
2. L’API `client` lance ensuite la conversation
3. Le serveur répondra en conséquence aux demandes du client.
4. L'application cliente se ferme si l’utilisateur entre le message `bye`. 
5. L'application serveur se ferme également lorsque le programme client se ferme.

### Les APIs (Application Programming Interface)

Les applications sont les suivantes :

- **API Serveur**: `socket_server.py`
- **API Client** : `socket_client.py`

Chaque application est codée en Python. 

#### API Serveur

`socket_server.py`

```python
import socket

def server_program():
    # get the hostname
    host = socket.gethostname()
    port = 5000  # initiate port no above 1024

    server_socket = socket.socket()  # get instance
    # look closely. The bind() function takes tuple as argument
    server_socket.bind((host, port))  # bind host address and port together

    # configure how many client the server can listen simultaneously
    server_socket.listen(2)
    conn, address = server_socket.accept()  # accept new connection
    print("Connection from: " + str(address))
    while True:
        # receive data stream. it won't accept data packet greater than 1024 bytes
        data = conn.recv(1024).decode()
        if not data:
            # if data is not received break
            break
        print("from connected user: " + str(data))
        data = input(' -> ')
        conn.send(data.encode())  # send data to the client

    conn.close()  # close the connection

if __name__ == '__main__':
    server_program()
```

Notre serveur fonctionne sur le port `5000` et il attendra la demande du client. `Python while loop` est utilisé pour exécuter le programme indéfiniment et continuer à attendre la demande du client. Il est recommandé d’utiliser l’adresse de port supérieure à `1024` car les numéros de port inférieur sont réservés aux protocoles internet standard.

#### API Client

`socket_client.py`

```python
import socket

def client_program():
    #host = socket.gethostname()  # as both code is running on same pc
    host = "172.19.0.10"  #IP Server, added from original code
    port = 5000  # socket server port number

    client_socket = socket.socket()  # instantiate
    client_socket.connect((host, port))  # connect to the server

    message = input(" -> ")  # take input

    while message.lower().strip() != 'bye':
        client_socket.send(message.encode())  # send message
        data = client_socket.recv(1024).decode()  # receive response

        print('Received from server: ' + data)  # show in terminal

        message = input(" -> ")  # again take input

    client_socket.close()  # close the connection

if __name__ == '__main__':
    client_program()
```

# 4. Livrables

## Partie 1

Vous devrez faire un fork du dépôt `Github` afin de récupérer les deux applications et de pouvoir envoyer vos modifications sur votre dépôt `GitHub` nommé `[Github_depot]\LIN2_TE` . Vous devrez mettre à disposition deux `Dockerfile` (un par application) ainsi qu'un fichier `docker-compose.yaml` pour le déploiement et le bulid automatique des deux applications. Vous devriez avoir une arborescence telle que :
```
[Github_depot]\LIN2_TE
.
├── api-server
│   ├── Dockerfile
│   └── socket_server.py
├── api-client
│   ├── Dockerfile
│   └── socket_client.py
└── docker-compose.yaml
└── readme.md
└── tp.md
```
### Dockerfile server

```
FROM python:3.11
WORKDIR /app/
COPY ./socket_server.py /app/
EXPOSE 5000
CMD [ "python", "./socket_server.py" ]
```

### Dockerfile client

```
FROM python:3.11
WORKDIR /app/
COPY ./socket_client.py /app/
EXPOSE 5000
CMD [ "python", "./socket_client.py" ]
```

### Docker Compose

```
version: '3'
networks:
  net:
    ipam:
      config:
      - subnet: 172.19.0.0/24
services:
  server:
    build:
      context: ./api-server/
      dockerfile: Dockerfile
    container_name: server
    restart: unless-stopped
    stdin_open: true
    tty: true
    networks:
      net:
        ipv4_address: 172.19.0.10
  client:
    build:
      context: ./api-client/
      dockerfile: Dockerfile
    container_name: client
    restart: unless-stopped
    ports:
      - 5000:5000
    stdin_open: true
    tty: true
    networks:
      net:
        ipv4_address: 172.19.0.20
    depends_on:
      - "server"
```



## Partie 2

Vous devrez ensuite publier vos images sur un compte `Dockerhub` que vous aurez préalablement créées. La version fonctionnelle doit être taguée en `latest`. L'adresse des images sur le `Dockerhub` devra figurer dans un champ `LABEL` de chaque Dockerfile, ayant pour valeur `hub_url`.

```
[DockerHub_perso]\lin2-server
[DockerHub_perso]\lin2-client
```
Un fichier `docker-compose-2.yaml` sera créé pour le déploiement des deux applications en utilisant les images présentes sur votre DockerHub personnel. Le fichier créé sera téléversé sur votre Github personnel.

```
[Github_depot]\LIN2_TE
.
├── api-server
│   ├── Dockerfile
│   └── socket_server.py
├── api-client
│   ├── Dockerfile
│   └── socket_client.py
└── docker-compose.yaml
└── docker-compose-2.yaml
└── readme.md
└── tp.md
```

### Docker compose 2

```
version: '3'
networks:
  net:
    ipam:
      config:
      - subnet: 172.19.0.0/24
services:
  server:
    image: "auggus/te-lin2-v2_server:v0.1"
    container_name: server
    restart: unless-stopped
    stdin_open: true
    tty: true
    networks:
      net:
        ipv4_address: 172.19.0.10
  client:
    image: "auggus/te-lin2-v2_client:v0.1"
    container_name: client
    restart: unless-stopped
    ports:
      - 5000:5000
    stdin_open: true
    tty: true
    networks:
      net:
        ipv4_address: 172.19.0.20
    depends_on:
      - "server"
```

## Commandes

```
Commandes : 

docker image build -t api-back:v0.1 . --no-cache
docker container run -p 8001:8001 api-back:v0.1
docker-compose up -d
docker-compose -f docker-compose-2.yaml up -d

docker attach 

SSH Key for Github : 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDrOkZ9jB4pPcglxB9D0BZdzoaaE/4WZe4mWqVlJCdRDELC+xwfU0L0fU8+Wl1aTg7xsQXjFsr29ov7V/qbK7pjmZN1wiTwro0gkyPoEcLKOj6EgtB81/kolcaIJsjIovmFf1+EgdFGnhRPbvCre5Y1FzG/Ro1UzNgtF251eBXsLPVbkRpo6tlazvG+q7JpPQX5j1YtNrqoedV/WFUwlfoALwoBhp0kONv8mf5Z/YVNjpXZKf86x3VjyMETzz2yRorFxOsIOKjpJbpJnKRepBZBNPvASuQ6gez83LYGeSbw1M/sJbB8keuwxxvlHKapYuRoCJlXl8Q8gDe+Ealwif4i4sVHmy5w1rVxY805/9EwisFwpEAaDVOCmDDl+M9IneWnl5/fxDHM/xqZaAHyBEYC1LckFA4OpVU6+wtOW+VC9Dik7JVxiI6L5gpw5Ed533pMzpBNboCWd0wHAaphJu7eEkpIn5nGaWD3FDOr8du538hwRK8h8P5wc+GG0YDM1StEotzWFytyJbiwY1oAhZSKkfm+lRK+N0HFbdaOep7eMmIfX/JRPaMgod1g5TL6lOKm2GDhMf8I34nCR/1tfFDPdboE0OkQtT0mq3cZ0RtvlqKvWEdaG9Naf0MGWPwOgaJ2b53xvSoTwyfbhDooGfVBREle1lEI/ZS6Ian05l4vDQ== David-Manuel.VAROSO-GOMES@cpnv.ch

docker tag te-lin2-v2_client:latest auggus/te-lin2-v2_client:v0.1
docker push auggus/te-lin2-v2_client:v0.1

docker tag te-lin2-v2_server:latest auggus/te-lin2-v2_server:v0.1
docker push auggus/te-lin2-v2_server:v0.1

 image: "auggus/te-lin2-v2_server:v0.1"
 image: "auggus/te-lin2-v2_client:v0.1"

### Supprimer tous les container et volumes 
sudo docker rm -vf $(sudo docker ps -aq)

### Supprimer tous les images 
sudo docker rmi -f $(sudo docker images -aq)

git add .
git status

git commit -m "tets"
git push
```



# 5. Evaluation

<aside>
⚠️ Aucune modification ne doit être faite sur le dépôt GitHub et le dépôt DockerHub après l'heure de fin de l'évaluation.
</aside>

```python
**Information à envoyer par mail à crn@cpnv.ch**
Adresse du dépôt GitHub :
Adresse de l'image Dockerhub :
```

Répondez aux questions du fichier `tp.md`

```
Nom : Varoso Gomes
Prénom : David Manuel
Classe : SI-T1b
Date : 26.01.2023

Quelle est la commande pour exécuter le fichier docker-compose.yaml ?
- docker-compose up -d

Quelle est la commande pour se connecter avec un terminal et interagir avec l'application serveur ?
- docker exec -it <container_name> bash

Quelle est la commande pour se connecter avec un terminal et interagir avec l'application cliente ?
- docker exec -it <container_name> bash

Quelle est la commande pour transformer le container serveur en une image ?
- docker container commit <container id>

Quelle est la commande pour pousser l'image précédemment créée sur dépôt personnel ?
- docker push auggus/[img-name]
```

## POC

1. Exécutez votre `docker-compose.yaml` afin de créer les deux containers et initialiser les deux applications, en premier l'application serveur suivit ensuite de l'application cliente.
2. Ouvrez une connexion terminale interactive sur l’API Serveur
3.  Ouvrez une connexion terminale interactive sur l’API Client
4. Tapez `Hello` dans le Client. Le message `Hello` doit apparaitre sur le Serveur.
5. Tapez `Hello` dans le Serveur. Le message `Hello` doit apparaitre sur le Client.
6. Taper `bye` dans le Client. L'application serveur et l'application cliente doivent se fermer.

# Notation

```python
Dépot Github                                   [ 2 pts]
- Tous les fichiers sont présents
- Fichier readme.md modifié et complété
Dépot Dockerhub                                [ 4 pts]
- 1 pt par image (2x)
- 1 pt pour le tag et la bonne version (2x)

DockerFile API server (présent + fonctionnel)  [ 2 pts]
DockerFile API client (présent + fonctionnel)  [ 2 pts]
docker-compose.yaml   (présent + fonctionnel)  [ 2 pts]
docker-compose-2.yaml (présent + fonctionnel)  [ 2 pts]

POC réussi :                                   [10 pts]
1.     (5 pts)
2.     (2 pts)
3.     (2 pts)
4.5.6  (1 pt )
------------------------------------------------------
TOTAL										  [24 pts]
------------------------------------------------------
           Note finale : Nombre de points / 24 * 5 + 1
```
