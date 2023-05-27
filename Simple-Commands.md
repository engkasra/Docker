For remove images

```bash
docker images -a
docker rm $(docker ps -a -f status=exited -q) #Remove all exited containers
docker rm $(docker ps -a -f status=exited -f status=created -q) #Remove containers using more than one filter
docker rmi $(docker images -a -q) #Remove all images
```
