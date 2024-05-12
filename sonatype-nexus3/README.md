### Deploying a Sonatype Nexus Repository Manager in Docker

Create working directory

```Bash
sudo mkdir /opt/nexus3
```

Copy compose.yaml into working directory

```Bash
sudo cp sonatype-nexus3/compose.yaml /opt/nexus3/
```

Create directory for persisting data

```Bash
sudo mkdir -p ./nexus-data
```

Fix permissions

```Bash
sudo chown -R 200 nexus-data/
```

>[!Note]
> To get user id in nexus container run:
> `docker exec <NEXUS_CONTAINER_ID> id`

Create and start Nexs container in background

```Bash
docker compose up -d
```

#### Check out Nexus state

Show application logs

```Bash
docker compose logs
```

List containers

```Bash
docker ps
```

##### References

> References
>
> 1. https://help.sonatype.com/en/installation-methods.html
> 2. https://cloud.croc.ru/blog/about-technologies/ustanovka-docker-repozitoriya/
> 3. https://stackoverflow.com/questions/51209615/permission-issues-in-nexus3-docker-container