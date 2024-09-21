# Docker: Next.js - Express - MongoDB

Docker deployment for a Next.js app with api's in Express.js and MongoDB as database using secure jwt cookies for authentication

## Learning Docker

```bash
docker ps # list of running containers
docker ps -a # list of all containers
docker images # list of used images
docker images -a # list of all images

docker container prune # deletes all not running containers
docker images prune # deletes all unused images

docker stop <container_id/container_name> # stops the container gracefully takes time
docker kill <container_id/container_name> # instantly kills/stops the container

docker rmi <image_name/image_id> # removes image from docker

docker network create <network_name> # create a new docker network
docker network ls # get the list of all the docker networks
docker network rm <network_name/network_id> # delete the network
docker volume create <volume_name> # create a new docker volume
docker volume ls # list of all docker volumes
docker volume rm <volume_name> # deletes a docker volume

docker exec -it <container_name> /bin/bash # puts us inside the bash shell of the container
docker exec -it busybox sh # puts us inside the bash shell of the container busybox

```

### creating an image for express server

_dockerfile_

```Dockerfile
FROM node:20 # or alpine-node:20   # it pulls a base image to start with

WORKDIR /usr/src/app  # sets a working directory in the container where the project files will be copied

COPY package* .  # copies the package.json and package-lock.json files first. Improves build time by taking advantage of caching this and the next step (since the next step is heavy on build time and resources) if the package.json has not changed.

RUN npm i    # installs the dependencies

COPY . .     # copies all the other project files to the working directory in the container

EXPOSE <port_number_to_expose>

# the commands above run when the container is built using the image

# the below command CMD runs when the container starts

CMD ["npm", "start"]    # runs when the container starts
```

_.dockerignore_

```.dockerignore
node_modules
```

bash command to build the image

```bash
docker build
 -t <image_name> # image tag
 . # directory where the Dockerfile is present
```

### running a container from an image for the first time

```bash
docker run
 -p <port_of_local_machine>:<port_of_docker_container> # to forward all the requests coming to the machine on the specified port to the container on the specified port
 --name <container_name> <image_name> # to organise the images otherwise a random id and name will be given to the image
```

### starting an already built container

```bash
docker start <container_name>
```

## adding volume to a database in docker

```bash
docker volume create <volume_name>
# docker volume ls   # to list all the volumes
# docker volume rm <volume_name>   # to delete the volume name
```

## adding network to the database

```bash
docker network create <network_name> # select network type to bridge
# docker network ls    # list all networks
# docker network rm <network_name>    # delete the specified network/s
```

### running the database container using the same network

```bash
docker build -p 27017:27017 --name mongodb-server --network <network_name> -v <volume_name> mongo # mongo is the image name of mongodb
```

### creating an express server to connect to the database on another container

_.index.js._

```node
const express = require("express");
const mongoose = require("mongoose");

const app = express();
const port = 8080;

mongoose.connect("mongodb:/mongodb-server:27017/myDatabase");
/* the "mongodb-server" in the connection string resolves to the ip address of the mongodb-server container using the same network as this express server container */

const EntrySchema = new mongoose.Schema({
  text: String,
  data: { type: Date, default: Date.now },
});

const Entry = mongoose.model("Entry", EntrySchema);

app.get("/", async (req, res) => {
  try {
    const entry = new Entry({ text: "new entry added" });
    await entry.save();
    res.send("New Entry Saved");
  } catch (err) {
    res.send(500).send("Error Occured");
  }
});

app.listen(port, () => {
  console.log("Example app listening on port", port);
});
```

_.Dockerfile._

```Dockerfile
FROM node:20

WORKDIR /usr/app/src

COPY package* .

RUN npm i

COPY . .

EXPOSE 8080

CMD ["npm","start"]
# or
# CMD ["npm","run","dev"] in development mode
```

_.dockerignore._

```dockerignore
node_modules
```

_.to build the image._

```bash
docker build -t backend-server .
```

;ls

to run the container from image

```bash
docker docker run -p 8080:8080 --name backend-sever --network <network_name>
```

now you can send get request to the server at http://localhost:8080/ and it will save the entry to the database
as the server will be able to connect to the database over the common network
