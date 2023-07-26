For remove images

```bash
docker images -a
docker rm $(docker ps -a -f status=exited -q) #Remove all exited containers
docker rm $(docker ps -a -f status=exited -f status=created -q) #Remove containers using more than one filter
docker rmi $(docker images -a -q) #Remove all images
sudo docker exec -it <ContainerID> curl -XPUT -k  -u "<ELASTIC_USER>:<ELASTIC_PASSWORD>" "http://localhost:9200/_license" -H "Content-Type: application/json" -d @/media/license.json #For Get Liecence File
``` 
