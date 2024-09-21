# Docker: Next.js - Express - MongoDB

Docker deployment for a Next.js app with api's in Express.js and MongoDB as database using secure jwt cookies for authentication

## Learning Docker

### creating an image for express server

*/dockerfile/*
```Dockerfile
FROM node:20 # or alpine-node:20   # it pulls a base image to start with

WORKDIR /usr/src/app  # sets a working directory in the container where the project files will be copied

COPY package* .  # copies the package.json and package-lock.json files first. Improves build time by taking advantage of caching this and the next step(since the next step is heavy on build time and resources) if the package.json has not changed. 

RUN npm i    # installs the dependencies

COPY . .     # copies all the other project files to the working directory in the container

# the commands above run when the container is built using the image

# the below command CMD runs when the container starts

CMD ["npm", "start"]    # runs when the container starts 
```

*/.dockerignore/*
```.dockerignore
node_modules
```

bash command to build the image 
```bash
docker build -t <image_name> . # where the . represents the current directory where the Dockerfile is present and -t for tagname (name of image)
```

### running a container from an image for the first time

```bash
docker run -p <port of local_machine>:<port of docker container to map with the local machine port> --name <container_name> <image_name>
```
