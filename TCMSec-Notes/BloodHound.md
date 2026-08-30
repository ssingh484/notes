```install-bloodhound
sudo pip install bloodhound
```

```start-db
sudo neo4j console
```
This provides an NPM server for Neo4J 
On login, change password, need to use in bloodhound

```run-bloodhound
sudo bloodhound
```

login to Neo4J database

Use a collector

```bloodhound-python
bloodhound-python -u <USER> -p <PASS> -d <domain> -c all -ns <Nameserver-IP>
```

# FOR DOCKER COMPOSE VERSION

Remember to stop and prune all containers and volumes before restarting stack for any new data/box

`sudo docker container prune -a`

`sudo docker volume prune -a`

