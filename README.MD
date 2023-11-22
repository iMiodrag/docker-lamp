# Docker Lamp with Ubuntu 22.04 & PHP 8.1 and mysql 8

Start by cloning this project
`git@github.com:iMiodrag/docker-lamp.git`

### Step 1 install
Create a file named **install_container.sh**

`docker build --build-arg="PHP_VERSION=8.1"  -t=imiodrag/lamp:latest-2204  -f ./2204/Dockerfile .`

### Step 2 start container

Make sure to replace values, also the first time you run it is going to return the mysql password, save that!

Create a file named **access_container.sh**

`docker run -i -t -p "8888:80" --name [PROVIDE_CONTAINER_NAME] -v ${PWD}/app:/app -v ${PWD}/mysql:/var/lib/mysql imiodrag/lamp:latest-2204`

### Step 3 access container

Create a file named access container, **make sure to replace the value with the value provided on step 2**

`docker exec -it [PROVIDE_CONTAINER_NAME] bash`

### Adding code

You can add the code inside the `app/public` folder

### Address

The default address is `localhost:8888` you can change this on step 2 if you like.