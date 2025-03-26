This project illustrates multiple apps interacting in Podman under Podman Compose with multiple Dockerfile.

**mysql-change-later-strapi**
is Strapi community backend app. Illustrating by Dockerfile to create image and run container of Podman and then we change the default SQLite DB to MySQL later upon successfully run the container.

Create this application by command $ *npx create-strapi-app@latest mysql-change-later-strapi --quickstart --js --install --skip-cloud --skip-db --no-run*


**Add-Dockerignore-Dockerfile** @Branch Name
Add *.dockerignore* & *Dockerfile* files to *mysql-change-later-strapi* application and then try to run container of Podman if image of Podman created by Dockerfile successfully.
This Docker / Podman Tutorial is according official site of Strapi : *https://docs.strapi.io/dev-docs/installation/docker*

Create Image by command $ *podman build --memory=8g -t image_mystrapi:latest -f Dockerfile*

Run Container by command $ *podman run -d --name strapi_container -p 1337:1337 localhost/image_mystrapi:latest*

Try *http://localhost:1337* at browser to verify if *container successfully running* and *image successfully created*.


**Dockercompose-with-database** @Branch Name
Now we create *docker-compose.yml* & .env files at root path of directory to create & run multiple services (apps), the multiple containers are running in an Pod of Podman, *docker-compose.yml* file is manipulating multiple *Dockerfile* files. 

This Tutorial we illustrate default DB platform of SQLite change to MySQL by *docker-compose.yml* and then running the multiple containers.

Remark: To process this you need to build Image of Strapi first, See tutorial at branch **Add-Dockerignore-Dockerfile**

Build & Uplink the Containers in an Pod by command $ *podman-compose up --build*