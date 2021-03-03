# Build and deploy a Drupal application via docker-compose
#Drupal is a CMS system similar to Wordpress. In this exercise first you must create a Dockerfile(by following instructions below) to build with docker-compose. Then create a docker-compose file where you deploy drupal with your image and Postgresql database by using its official image(so no Dockerfile for postgresql)
#Dockerfile
- First you need to build a custom Dockerfile in this directory, `FROM drupal:8.2`
- Then RUN apt package manager command to install git: `apt-get update && apt-get install -y git`
- Remember to cleanup after your apt install with `rm -rf /var/lib/apt/lists/*` and use `\` and `&&` properly. You can find examples of them in drupal official image. More on this below under Compose file.
- Then change `WORKDIR /var/www/html/themes`
- Then use git to clone the theme with: `RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git`
- Combine that line with this line, as we need to change permissions on files and don't want to use another image layer to do that (it creates size bloat). This drupal container runs as www-data user but the build actually runs as root, so often we have to do things like `chown` to change file owners to the proper user: `chown -R www-data:www-data bootstrap`. Remember the fist line needs a `\` at end to signify the next line is included in the command, and at start of next line you should have `&&` to signify "if first command succeeds then also run this command"
- Then, just to be safe, change the working directory back to its default (from drupal image) at `/var/www/html`
# 
``` FROM drupal:8.2
RUN apt-get update && apt-get install -y git && \
 rm -rf /var/lib/apt/lists/*
WORKDIR /var/www/html/themes
RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git && chown -R www-data:www-data bootstrap
WORKDIR /var/www/html          
```

# Create a new directory
``` mkdir my_drupal```

``` cd my_drupal```
# Compose File
- We're going to build a custom image in this compose file for drupal service.  
- Create a drupal container(in docker-compose containers called "services") called `drupal`. We want to build the default Dockerfile in this directory by adding `build: .` to the `drupal` service. When we add a build + image value to a compose service, it knows to use the image name to write to in our image cache, rather then pull from Docker Hub.
- For Postgresql application use the official image `postgres:10`, check from Dockerhub what parameters are necessary to launch this image.  Also add a volume for `drupal-data:/var/lib/postgresql/data` so the database will persist across Compose restarts.
## 
Create a docker-compose.yml
``` docker-compose.yml```

```
version: '3.3'
services:
  app:
    build: .
    image: drupal:8.2
    ports:
      - 8089:80
    restart: always

  postgres:
    image: postgres:10
    environment:
      POSTGRES_PASSWORD: password
    volumes:
        - db_data:/var/lib/postgresql/data
    restart: always

volumes:
  db_data:

  ```
  # Start Containers, Configure Drupal
- Launch containers with docker-compose, you will see the first time docker-compose will build the image. But after this, eeven if you make changes to your Dockerfile, docker-compose can't be aware of it. So if you need to rebuild the application, you must launch the command to build the immage: `docker-compose build`, then you can launch containers.
- Check their status with `docker-compose ps` and check application logs with `docker-compose logs`.
- After website comes up, click on `Appearance` in top bar, and notice a new theme called `Bootstrap` is there. That's the one we added with our custom Dockerfile.
- Click `Install and set as default`. Then click `Back to site` (in top left) and the website interface should look different. You've successfully installed and activated a new theme in your own custom image without installing anything on your host other then Docker!
- If you exit (ctrl-c) and then `docker-compose down` it will delete containers, but not the volumes, so on next `docker-compose up` everything will be as it was.
- To totally clean up volumes, add `-v` to `down` command.

