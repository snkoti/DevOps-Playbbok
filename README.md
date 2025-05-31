# Docker Lab

### Docker Installation (AWS Linux Instance) via docker.io

``` bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo apt-get install docker-compose -y
```

### Docker installation via apt repository (Recommended)
1. Install Docker -- https://docs.docker.com/engine/install/ubuntu/

2. Add user to the docker group - < sudo usermod -aG docker $USER >  [ exit and ssh back to to apply the changes. This step is important as it refreshes your session and grants your user the new group permissions]

3. docker --version

4. docker compose version

5. Expose port 3000 in AWS Security Group for the App


## PROJECT-1 ( Containerize NodeJS APP)

## LAB-1 Get the app

Before we can run the application, we need to get the application source code onto our machine.

1. Clone theÂ repo --> getting-started using the following command:
    
    ```jsx
      git clone https://github.com/amigo-nishant/getting-started.git
    ```
    
2. View the contents of the cloned repository. We should see the following files and sub-directories.
    
    ```jsx
    â”œâ”€â”€ getting-started-app/
    â”‚ â”œâ”€â”€ package.json
    â”‚ â”œâ”€â”€ README.md
    â”‚ â”œâ”€â”€ spec/
    â”‚ â”œâ”€â”€ src/
    â”‚ â””â”€â”€ yarn.lock
    ```
    

**Build the appâ€™s image**

To build the image, We'll need to use a Dockerfile. A Dockerfile is simply a text-based file with no file extension that contains a script of instructions. Docker uses this script to build a container image.

1. In theÂ `getting-started-app`Â directory, the same location as theÂ `package.json`Â file, create a file namedÂ `Dockerfile`. 
2.  We can use the following commands to create a Dockerfile based on our operating system.
    
    ---
    
    Create an empty file namedÂ `Dockerfile`.
    
    ```jsx
      cd getting-started/
      touch Dockerfile
    ```
    
    ---
    
3. Using a text editor or code editor, add the following contents to the Dockerfile:
    
    ```jsx
 
    FROM node:18-alpine
    WORKDIR /app
    COPY . .
    RUN yarn install --production
    CMD ["node", "src/index.js"]
    EXPOSE 3000
    ```
    
4. Build the image using the following commands:
    
    ```jsx
      docker build -t getting-started .
    ```
    
    TheÂ `docker build`Â command uses the Dockerfile to build a new image. You might have noticed that Docker downloaded a lot of "layers". This is because you instructed the builder that you wanted to start from theÂ `node:18-alpine`Â image. But, since you didn't have that on your machine, Docker needed to download the image.
    
    After Docker downloaded the image, the instructions from the Dockerfile copied in your application and usedÂ `yarn`Â to install your application's dependencies. TheÂ `CMD`Â directive specifies the default command to run when starting a container from this image.
    
    Finally, theÂ `-t`Â flag tags your image. Think of this as a human-readable name for the final image. Since you named the imageÂ `getting-started`, you can refer to that image when you run a container.
    
    TheÂ `.`Â at the end of theÂ `docker build`Â command tells Docker that it should look for theÂ `Dockerfile`Â in the current directory.
    

## Start the app container

Now that we have an image, we can run the application in a container using theÂ `docker run`Â command.

1. Run our container using theÂ `docker run`Â command and specify the name of the image we just created:
    
    ```jsx
      docker run -dp 3000:3000 getting-started
    ```
    
    TheÂ `-d`Â flag (short forÂ `--detach`) runs the container in the background. This means that Docker starts your container and returns you to the terminal prompt. You can verify that a container is running by viewing it in Docker Dashboard underÂ **Containers**, or by runningÂ `docker ps`Â in the terminal.
    
    TheÂ `-p`Â flag (short forÂ `--publish`) creates a port mapping between the host and the container. TheÂ `-p`Â flag takes a string value in the format ofÂ `HOST:CONTAINER`, whereÂ `HOST`Â is the address on the host, andÂ `CONTAINER`Â is the port on the container. The command publishes the container's port 3000 toÂ `3000`Â (`localhost:3000`) on the host. Without the port mapping, you wouldn't be able to access the application from the host.
    
2. After a few seconds, open your web browser toÂ [http://localhost:3000](http://localhost:3000/). You should see your app.
    
    
3. Add an item or two and see that it works as we expect. We can mark items as complete and remove them. Our frontend is successfully storing items in the backend.

At this point, we have a running todo list manager with a few items.

If you take a quick look at your containers, you should see at least one container running that's using theÂ `getting-started`Â image and on portÂ `3000`. To see your containers, you can use the CLI or Docker Desktop's graphical interface.

CLIÂ Docker Desktop

---

Run the followingÂ `docker ps`Â command in a terminal to list your containers.

```jsx
  docker ps
```

Output similar to the following should appear.

```jsx
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
df784548666d        getting-started     "docker-entrypoint.sâ€¦"   2 minutes ago       Up 2 minutes        127.0.0.1:3000->3000/tcp   priceless_mcclintock
```

---

## LAB-2 Update the source code -

1. In theÂ `src/static/js/app.js`Â file, update line 56 to use the new empty text.
    
    ```jsx
    <p className="text-center">No items yet! Add one above!</p>
    <p className="text-center">You have no todo items yet! Add one above!</p>
    ```
    
2. Build your updated version of the image, using theÂ `docker build`Â command.
    
    ```jsx
      docker build -t getting-started .
    ```
    
3. Start a new container using the updated code.
    
    ```jsx
      docker run -dp 3000:3000 getting-started
    ```
    
4. You probably saw an error like this:

```jsx
docker: Error response from daemon: driver failed programming external connectivity on endpoint laughing_burnell 
(bb242b2ca4d67eba76e79474fb36bb5125708ebdabd7f45c8eaf16caaabde9dd): Bind for 3000 failed: port is already allocated.
```

The error occurred because we aren't able to start the new container while our old container is still running. The reason is that the old container is already using the host's port 3000 and only one process on the machine (containers included) can listen to a specific port. To fix this, we need to remove the old container.

## Remove the old container

To remove a container, we first need to stop it. Once it has stopped, we can remove it. We can remove the old container using the CLI or Docker Desktop's graphical interface. 

1. Get the ID of the container by using theÂ `docker ps`Â command.

---

```jsx
  docker ps
```

2. Use theÂ docker stopÂ command to stop the container. ReplaceÂ <the-container-id>Â with the ID fromÂ docker ps.

```jsx
  docker stop <the-container-id>
```

3. Once the container has stopped, we can remove it by using theÂ docker rmÂ command.

```jsx
  docker rm <the-container-id>
```

**NOTE:**
We can stop and remove a container in a single command by adding theÂ forceÂ flag to theÂ docker rmÂ command. For example:Â `docker rm -f <the-container-id>`

```jsx
docker rm -f <the-container-id>
```

---

### Start the updated app container

1. Now, start your updated app using theÂ `docker run`Â command.
    
    ```jsx
      docker run -dp 3000:3000 getting-started
    ```
    
2. Refresh your browserÂ and you should see your updated help text.

## Summary

In this section, we learned how to update and rebuild a container, as well as how to stop and remove a container.

## LAB-3 Create a repository

To push an image, we first need to create a repository on Docker Hub.

1. [Sign up](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade)Â or Sign in toÂ [Docker Hub](https://hub.docker.com/).
2. Select theÂ **Create Repository**Â button.
3. For the repository name, useÂ `getting-started`. Make sure theÂ **Visibility**Â isÂ **Public**.
4. SelectÂ **Create**.

## Push the image

1. In the command line, run theÂ `docker push`Â command that you see on Docker Hub. Note that your command will have your Docker ID, not "docker". For example,Â `docker push YOUR-USER-NAME/getting-started`.
    
    ```jsx
      docker push docker/getting-started
    ```
    The push refers to repository [docker.io/docker/getting-started]
    An image does not exist locally with the tag: docker/getting-started
    
    Why did it fail? The push command was looking for an image namedÂ `docker/getting-started`, but didn't find one. If you runÂ `docker image ls`, you won't see one either.
    
    To fix this, you need to tag your existing image you've built to give it another name.
    
2. Use theÂ `docker tag`Â command to give theÂ `getting-started`Â image a new name. ReplaceÂ `YOUR-USER-NAME`Â with your Docker ID.
    
    ```jsx
      docker tag getting-started YOUR-USER-NAME/getting-started
    ```
    
3. Now run theÂ `docker push`Â command again. If you're copying the value from Docker Hub, you can drop theÂ `tagname`Â part, as you didn't add a tag to the image name. If you don't specify a tag, Docker uses a tag calledÂ `latest`.
    
    ```jsx
      docker push YOUR-USER-NAME/getting-started
    ```
    

## Run the image in new instance

Now that your image has been built and pushed into a registry, try running your app on a brand new instance that has never seen this container image. 

## LAB-4 **Persist the DB**

## Persist the Data

By default, the todo app stores its data in a SQLite database atÂ `/etc/todos/todo.db`Â in the container's filesystem. 

### Create a volume and start the container

You can create the volume and start the container using the CLI or Docker Desktop's graphical interface.

---

1. Create a volume by using theÂ `docker volume create`Â command.

```jsx
  docker volume create todo-db
```

2. Start the todo app container, but add theÂ `--mount`Â option to specify a volume mount. Give the volume a name, and mount it toÂ `/etc/todos`Â in the container, which captures all files created at the path. 

```jsx
  docker run -dp 3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

---

### Verify that the data persists

1. Once the container starts up, open the app and add a few items to your todo list.
2. Stop and remove the container for the todo app. Use Docker Desktop orÂ `docker ps`Â to get the ID and thenÂ `docker rm -f <id>`Â to remove it.
3. Start a new container using the previous steps.
4. Open the app. You should see your items still in your list.
5. Go ahead and remove the container when you're done checking out your list.

## Dive into the Volume

A lot of people frequently ask "Where is Docker storing my data when I use a volume?" If you want to know, you can use theÂ `docker volume inspect`Â command.

```jsx
  docker volume inspect todo-db
[
    {
        "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

TheÂ `Mountpoint`Â is the actual location of the data on the disk. Note that on most machines, you will need to have root access to access this directory from the host.

## Summary

In this section, you learned how to persist container data.

# **LAB - 5 Use Bind mounts**

So far we used a volume mount to persist the data in your database. A volume mount is a great choice when you need somewhere persistent to store your application data.

A bind mount is another type of mount, which lets you share a directory from the host's filesystem into the container. When working on an application, you can use a bind mount to mount source code into the container. The container sees the changes you make to the code immediately, as soon as you save a file. This means that you can run processes in the container that watch for filesystem changes and respond to them.

## Bind Mounts

Before looking at how you can use bind mounts for developing your application, you can run a quick experiment to get a practical understanding of how bind mounts work.

1. Verify that yourÂ `getting-started-app`Â directory is in a directory defined in Docker Desktop's file sharing setting. 
2. Open a terminal and change directory to theÂ `getting-started-app`Â directory.
3. Run the following command to startÂ `bash`Â in anÂ `ubuntu`Â container with a bind mount.
    
    ---
    
    ```jsx
      docker run -it --mount type=bind,src="$(pwd)",target=/src ubuntu bash
    ```
    
    ---
    
    TheÂ `--mount`Â option tells Docker to create a bind mount, whereÂ `src`Â is the current working directory on your host machine (`getting-started-app`), andÂ `target`Â is where that directory should appear inside the container (`/src`).
    
4. After running the command, Docker starts an interactiveÂ `bash`Â session in the root directory of the container's filesystem.
    
    ```jsx
    root@ac1237fad8db:/# pwd
    /
    root@ac1237fad8db:/# ls
    bin   dev  home  media  opt   root  sbin  srv  tmp  var
    boot  etc  lib   mnt    proc  run   src   sys  usr
    ```
    
5. Change directory to theÂ `src`Â directory.
    
    This is the directory that you mounted when starting the container. Listing the contents of this directory displays the same files as in theÂ `getting-started-app`Â directory on your host machine.
    
    ```jsx
    root@ac1237fad8db:/# cd src
    root@ac1237fad8db:/src# ls
    Dockerfile  node_modules  package.json  spec  src  yarn.lock
    ```
    
6. Create a new file namedÂ `myfile.txt`.
    
    ```jsx
    root@ac1237fad8db:/src# touch myfile.txt
    root@ac1237fad8db:/src# ls
    Dockerfile  myfile.txt  node_modules  package.json  spec  src  yarn.lock
    ```
    
7. Open theÂ `getting-started-app`Â directory on the host and observe that theÂ `myfile.txt`Â file is in the directory.
    
    ```jsx
    â”œâ”€â”€ getting-started-app/
    â”‚ â”œâ”€â”€ Dockerfile
    â”‚ â”œâ”€â”€ myfile.txt
    â”‚ â”œâ”€â”€ node_modules/
    â”‚ â”œâ”€â”€ package.json
    â”‚ â”œâ”€â”€ spec/
    â”‚ â”œâ”€â”€ src/
    â”‚ â””â”€â”€ yarn.lock
    ```
    
8. From the host, delete theÂ `myfile.txt`Â file.
9. In the container, list the contents of theÂ `app`Â directory once more. Observe that the file is now gone.
    
    ```jsx
    root@ac1237fad8db:/src# ls
    Dockerfile  node_modules  package.json  spec  src  yarn.lock
    ```
    
10. Stop the interactive container session withÂ `Ctrl`Â +Â `D`.

That's all for a brief introduction to bind mounts. This procedure demonstrated how files are shared between the host and the container, and how changes are immediately reflected on both sides. Now you can use bind mounts to develop software.

# **LAB - 6 Multi container apps**

Up to this point, we've been working with single container apps. But, now we will add MySQL to the application stack. 

## **Container Networking**

Remember that containers, by default, run in isolation and don't know anything about other processes or containers on the same machine. So, how do you allow one container to talk to another? The answer is networking. If you place the two containers on the same network, they can talk to each other.

## **Start MySQL**

In the following steps, we'll create the network first and then attach the MySQL container at startup.

1. Create the network.
    
    ```jsx
      docker network create todo-app
    ```
    
2. Start a MySQL container and attach it to the network. You're also going to define a few environment variables that the database will use to initialize the database. 
    
    ---
    
    ```jsx
      docker run -d \
        --network todo-app --network-alias mysql \
        -v todo-mysql-data:/var/lib/mysql \
        -e MYSQL_ROOT_PASSWORD=secret \
        -e MYSQL_DATABASE=todos \
        mysql:8.0
    ```
    
    ---
    
    In the previous command, you can see theÂ `--network-alias`Â flag. In a later section, you'll learn more about this flag.
    
    You'll notice a volume namedÂ `todo-mysql-data`Â in the above command that is mounted atÂ `/var/lib/mysql`, which is where MySQL stores its data. However, you never ran aÂ `docker volume create`Â command. Docker recognizes you want to use a named volume and creates one automatically for you.
    
3. To confirm you have the database up and running, connect to the database and verify that it connects.
    
    ```jsx
      docker exec -it <mysql-container-id> mysql -u root -p
    ```
    
    When the password prompt comes up, type inÂ `secret`. In the MySQL shell, list the databases and verify you see theÂ `todos`Â database.
    
    ```jsx
    mysql> SHOW DATABASES;
    ```
    
    You should see output that looks like this:
    
    ```jsx
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    | todos              |
    +--------------------+
    5 rows in set (0.00 sec)
    ```
    
4. Exit the MySQL shell to return to the shell on your machine.
    
    ```jsx
    mysql> exit
    ```
    
    You now have aÂ `todos`Â database and it's ready for you to use.
    

## Run your app with MySQL

The todo app supports the setting of a few environment variables to specify MySQL connection settings. They are:

- `MYSQL_HOST`Â - the hostname for the running MySQL server
- `MYSQL_USER`Â - the username to use for the connection
- `MYSQL_PASSWORD`Â - the password to use for the connection
- `MYSQL_DB`Â - the database to use once connected

You can now start your dev-ready container.

1. Specify each of the previous environment variables, as well as connect the container to your app network. Make sure that you are in theÂ `getting-started-app`Â directory when you run this command.
    
    ---
    
    ```jsx
      docker run -dp 3000:3000 \
      -w /app -v "$(pwd):/app" \
      --network todo-app \
      -e MYSQL_HOST=mysql \
      -e MYSQL_USER=root \
      -e MYSQL_PASSWORD=secret \
      -e MYSQL_DB=todos \
      node:18-alpine \
      sh -c "yarn install && yarn run dev"
    ```
    
    ---
    
2. If you look at the logs for the container (`docker logs -f <container-id>`), you should see a message similar to the following, which indicates it's using the mysql database.
    
    ```jsx
     nodemon src/index.js
    [nodemon] 2.0.20
    [nodemon] to restart at any time, enter `rs`
    [nodemon] watching dir(s): *.*
    [nodemon] starting `node src/index.js`
    Connected to mysql db at host mysql
    Listening on port 3000
    ```
    
3. Open the app in your browser and add a few items to your todo list.
4. Connect to the mysql database and prove that the items are being written to the database. Remember, the password isÂ `secret`.
    
    ```jsx
      docker exec -it <mysql-container-id> mysql -p todos
    ```
    
    And in the mysql shell, run the following:
    
    ```jsx
    mysql> select * from todo_items;
    +--------------------------------------+--------------------+-----------+
    | id                                   | name               | completed |
    +--------------------------------------+--------------------+-----------+
    | c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
    | 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
    +--------------------------------------+--------------------+-----------+
    ```
    
    Your table will look different because it has your items. But, you should see them stored there.
    

## Summary

At this point, you have an application that now stores its data in an external database running in a separate container. You learned a little bit about container networking and service discovery using DNS.

# **LAB - 7 Use Docker Compose**

Docker-compose is a tool that helps you define and share multi-container applications. With Compose, you can create a YAML file to define the services and with a single command, you can spin everything up or tear it all down.

## Create the Compose File

In theÂ `getting-started-app`Â directory, create a file namedÂ `compose.yaml`.

```jsx
â”œâ”€â”€ getting-started-app/
â”‚ â”œâ”€â”€ Dockerfile
â”‚ â”œâ”€â”€ compose.yaml
â”‚ â”œâ”€â”€ node_modules/
â”‚ â”œâ”€â”€ package.json
â”‚ â”œâ”€â”€ spec/
â”‚ â”œâ”€â”€ src/
â”‚ â””â”€â”€ yarn.lock
```

## Define the app Service

Previously we used the following command to start the application service.

```jsx
  docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:18-alpine \
  sh -c "yarn install && yarn run dev"
```

Below is the yaml compose file.

```jsx
services:
  app:
    image: node:18-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
```

### Define the MySQL Service

Now, it's time to define the MySQL service. The command that you used for that container was the following:

```jsx
  docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:8.0
```

Below is the yaml compose file.

```jsx
services:
  app:
    # The app service definition
  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

At this point, our completeÂ `compose.yaml`Â should look like this:

```jsx
services:
  app:
    image: node:18-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
    depends_on:
      - mysql

  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

## Run the application stack

Now that we have ourÂ `compose.yaml`Â file, we can start our application.

1. Make sure no other copies of the containers are running first. UseÂ `docker ps`Â to list the containers andÂ `docker rm -f <ids>`Â to remove them.
2. Start up the application stack using theÂ `docker compose up`Â command. Add theÂ `d`Â flag to run everything in the background.
    
    ```jsx
      docker compose up -d
    ```
    
    When you run the previous command, you should see output like the following:
    
    ```jsx
    Creating network "app_default" with the default driver
    Creating volume "app_todo-mysql-data" with default driver
    Creating app_app_1   ... done
    Creating app_mysql_1 ... done
    ```
    
    You'll notice that Docker Compose created the volume as well as a network. By default, Docker Compose automatically creates a network specifically for the application stack (which is why you didn't define one in the Compose file).
    
3. Look at the logs using theÂ `docker compose logs -f`Â command. You'll see the logs from each of the services interleaved into a single stream. This is incredibly useful when you want to watch for timing-related issues. TheÂ `f`Â flag follows the log, so will give you live output as it's generated.
    
    If you have run the command already, you'll see output that looks like this:
    
    ```jsx
    mysql_1  | 2019-10-03T03:07:16.083639Z 0 [Note] mysqld: ready for connections.
    mysql_1  | Version: '8.0.31'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
    app_1    | Connected to mysql db at host mysql
    app_1    | Listening on port 3000
    ```
    
    The service name is displayed at the beginning of the line (often colored) to help distinguish messages. If you want to view the logs for a specific service, you can add the service name to the end of the logs command (for example,Â `docker compose logs -f app`).
    
4. At this point, you should be able to open your app in your browser onÂ [http://localhost:3000](http://localhost:3000/)Â and see it running.

## Bring down the micro-services

When you're ready to tear it all down, simply runÂ `docker compose down`Â or hit the trash can on the Docker Dashboard for the entire app. The containers will stop and the network will be removed.

```jsx
  docker compose down
```

## Summary

In this section, you learned about Docker Compose and how it helps you simplify the way you define and share multi-service applications.

