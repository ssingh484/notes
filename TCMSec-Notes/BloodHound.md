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