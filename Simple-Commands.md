**Some Useful Commands**
```bash
docker images -a
docker rm $(docker ps -a -f status=exited -q) #Remove all exited containers
docker rm $(docker ps -a -f status=exited -f status=created -q) #Remove containers using more than one filter
docker rmi $(docker images -a -q) #Remove all images
sudo docker exec -it <ContainerID> curl -XPUT -k  -u "<ELASTIC_USER>:<ELASTIC_PASSWORD>" "http://localhost:9200/_license" -H "Content-Type: application/json" -d @/media/license.json #For Get Liecence File
``` 
**How to get a Docker container's IP address from the host**
```bash
docker inspect <container ID>
docker inspect <container id> | grep "IPAddress"
``` 
**How to get docker-compose to always re-create containers from fresh images?**
```bash
docker-compose down && docker-compose build --no-cache && docker-compose up
```
**How to Clear Docker Cache?**
```bash
docker image prune -a -f #This command will remove all the unused images, including their intermediate layers. Be careful when using this command as it can remove images that you may still need.
docker sudo builder prune -a #This command will remove any cached layers for images that have been removed with “docker rmi” but for which caches are still present. It’s important to note that this command only removes cache for images that have been removed with “docker rmi” and are not visible with “docker images –all”.
docker system prune -a #The easiest way to clear the Docker cache, as well as other unused resources like stopped containers and dangling images. The -a flag removes all unused images, not just the dangling ones. Confirm the action by typing ‘y’ when prompted.
```
**Docker run with --rm flag**
The docker run command is docker's standard tool to help you start and run your containers. The --rm flag is there to tell the Docker Daemon to clean up the container and remove the file system after the container exits
