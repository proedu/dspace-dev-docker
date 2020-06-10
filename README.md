# dspace-dev-docker

DSpace instant development environment using Docker Compose (Currently for Linux)

# What?

This is currently a proof of concept. It aims at offering an easy to install, productivity focused, DSpace development 
environment.

# Requirements
- Install [Docker](https://docs.docker.com/engine/installation/)
- Install [Docker Compose](https://docs.docker.com/compose/install/) (Make sure to install the binaries, not the container)
- Have ports 8080 (tomcat), 5432 (postgresql), 1043 and 8000 (remote debugging) open. Otherwise, you can modify the mappings
  in docker-compose.yml file to use whichever ports you prefer.
- Git
- This currently works on Linux (tested in Ubuntu). Docker has announced better integration with OsX and Windows, allowing better volume mounts, which should eventually make this useful in other OSs.

# Launching
- Clone this repo 

        git clone https://github.com/pmarrone/dspace-dev-docker.git
        cd dspace-dev-docker

 - Add a dspace-src folder, where your DSpace code will reside. Also, add m2-repo and dspace-build folders.

        git clone -b dspace-6.0 https://github.com/DSpace/DSpace.git dspace-src
        mkdir m2-repo dspace-build
        
You should end up with the folder structure that dspace-dev-docker expects:

      dspace-dev-docker
      |-- dspace-src
      |-- dspace-build
      +-- m2-repo
      
> If you were to change this names, be aware that changing dspace-dev-docker to something else will affect the 'attach' command later on. If you change dspace-src, dspace-build or m2-repo to a different name, be sure to check those names in the docker-compose.yml file under the volumes settings so that they are mapped correctly.

<!-- -->
> If you fail to create dspace-src, dspace-build and m2-repo folders, Docker will create them for you on startup, but belonging to the ROOT user. Make sure to change ownership of this folders to your user (e.g., sudo chown -R youruser:youruser dspace-src dspace-build m2-repo) or compilation and ant tasks will fail

<!-- -->
> When you run ant tasks, this container expects dspace to be installed on the /srv/dspace folder. Edit your local.cfg file in the dspace-src folder so that dspace.install.dir=/srv/dspace. Otherwise, running fresh_install will fail. 

<!-- -->
> You will need to change the db.url property in the local.cfg file to ```db.url=jdbc:postgresql://postgres:5432/dspace``` to make the database connection work (notice postgres is used instead of localhost). The expected DB credentials are dspace:dspace for the dspace database.

 - Launch Docker compose and let the magic happen. It takes a while to download the first time you run it.

         docker-compose up -d

 - Once launched, you should be able to attach to the container's bash process. This will get you into a 'developer' account

        docker attach dspacedevdocker_dspace-dev_1

# Doing things

Once inside the container, you can do dspace things, as packaging the project. The container should start on the dspace-src folder. E.g: lets compile DSpace with Mirage2 enabled

    mvn -Dmirage2.on=true -Dmirage2.deps.included=false package

Once compiled, the 'task' alias is available. This basically runs ant using the build.xml found in /srv/dspace-src/dspace/target/dspace-installer/build.xml, so that you don't have to move back and forth to that folder. So, you can run from anywhere

    task fresh_install
    
Also, an alias to the dspace binary is available, so you can run from anywhere

    dspace create-administrator

The tomcat alias is also available, to quickly launch and stop tomcat. The catalina.out alias lauches 'tail' to follow tomcat's log file.

    tomcat start && catalina.out

Since you bound the container's 8080 port to your host, once launched, you can access DSpace xml UI from http://localhost:8080/xmlui/

Tomcat's manager app is also availabe with user: *admin* password: *admin* from http://localhost:8080/manager/
so that you can restart your webapps without restarting tomcat. 
> PSI probe is also installed, but currently broken. I'll get to this eventually

One of the most important benefits of this container right now is that it uses the [DCEVM](https://dcevm.github.io/) patched JVM with [HotswapAgent](http://www.hotswapagent.org/) installed. Just start tomcat with the jpda option

    tomcat jpda start
    
This should get you a tomcat instance running in debugging mode with enhanced hotswapping capabilities. This debugger will be running on port 1043, which is mapped to the host's 1043 port.
Check https://github.com/HotswapProjects/HotswapAgent/wiki/Intellij-IDEA-setup to get an idea of how to set hot-swapping in your IDE, and http://www.javaranch.com/journal/200408/DebuggingServer-sideCode.html to setup remote debugging. This examples
work on IntelliJ, but the setup should be somewhat similar in other IDEs.
