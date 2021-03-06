# Installation and configuration



To use the [miner](https://github.com/ropenscilabs/miner) package, you'll need a Minecraft server that is using the [RaspberryJuice](https://dev.bukkit.org/projects/raspberryjuice) plugin. You can use [Spigot](https://www.spigotmc.org) to set up your own server, even locally on your machine. The installation process doesn't take too long, but it helps to have some _command line_ experience.

## Mac OS X

Installing stuff on a Mac.

## Windows

Installing stuff in Windows.

## Linux

These instructions describe how to set up a Minecraft Server on Linux with the Raspberry Juice plugin. Once installed, you can connect to the server with the Microsoft game and with R via the [miner](https://github.com/ropenscilabs/miner) package. If you are new to Minecraft, you will first have to make a one time purchase of a Minecraft license.

### Install

First, make sure you have installed [Java](https://www.java.com/en/download/help/linux_x64_install.xml). Then make a directory for Minecraft and change into it.


```bash
mkdir ~/minecraft
cd ~/minecraft
```

Download `Buildtools.jar` from [Spigot](https://www.spigotmc.org/wiki/spigot-installation/), a popular site for Minecraft server downloads. You will use the Buildtools program to complete the install. Run the `jar` file. This step will fail to start the server but will successfully create the plugin directory and the EULA. 


```bash
wget https://hub.spigotmc.org/jenkins/job/BuildTools/lastSuccessfulBuild/artifact/target/BuildTools.jar
sudo java -jar BuildTools.jar --rev 1.12
java -jar -Xms1024M -Xmx2048M spigot-1.12.jar nogui
```

Edit the file `eula.txt` so that the line `eula=false` is instead `eula=true`

Start up the server again. This will take a while (because it's building the world), but not as long as the initial compiling.


```bash
java -jar -Xms1024M -Xmx2048M spigot-1.12.jar nogui
```

Your Minecraft server should now be running. Open your Minecraft game on your desktop and connect to your server IP in multiplayer mode. You can make a player an operator by typing `op <playername>` into the server prompt. When you are finished playing type `stop` in the server prompt to stop the server.

### Connect with miner

You can use the [RaspberryJuice plugin](https://www.spigotmc.org/resources/raspberryjuice.22724/) to connect to your Minecraft Server via the [miner](https://github.com/ropenscilabs/miner) package. Download the plugin by visiting [its page](https://www.spigotmc.org/resources/raspberryjuice.22724/) and clicking the "Download Now" button in the upper-right. Move this `.jar` file to the `plugins` directory.


```bash
sudo wget https://github.com/zhuowei/RaspberryJuice/raw/master/jars/raspberryjuice-1.9.1.jar
mv raspberryjuice-1.9.1.jar ~/minecraft/plugins
```

Connect to your server from R using `mc_connect("<server-ip>")`. Test your connection by retrieving your player's location.


```r
library(miner)
mc_connect("<server-ip>")
getPlayerIds()
```

### Configure

The `~/minecraft/server.properties` file contains a list of configuration parameters for your Minecraft server. You will probably want to set `gamemode=1` and `force-gamemode=true`. If you want to create a superflat world also set `level-type=FLAT`.

```
gamemode=1
force-gamemode=true
level-type=FLAT
```

If you want to run Minecraft in the background, then you can create a simple `start.sh` script:


```bash
#!/bin/sh
java -Xms512M -Xmx1G -XX:+UseConcMarkSweepGC -jar spigot-1.12.jar
```

Then make it an executable, and run it with `nohup`: 


```bash
chmod +x start.sh
nohup ./start.sh
```

If you need to use a different port, use the `-p` option. ([See other options](https://www.spigotmc.org/wiki/start-up-parameters/).)


```bash
java -jar -Xms1024M -Xmx2048M spigot-1.12.jar -p25566 nogui
```

If you're having a hard time connecting, verify that your ports are open. The standard port for Minecraft is `25565`. The standard port for the [miner](https://github.com/ropenscilabs/miner) package is `4711`.


```bash
telnet <server-ip> 25565
telnet <server-ip> 4711
```


## Docker

### What is Docker?

Docker is a program that runs on runs on Linux, OSX or Windows to set up a tiny operating system on your compute, like having a computer in your computer. The advantage of this is that it can save a lot of bother troubleshooting problem relating to the unique configuration details of your computer. With Docker we can set up an isolated operating system on your computer that is already equipped up with a Minecraft server and the various dependencies described above, so we don't have to worry to about installing and configuring each item. Using a Docker container can take a lot of the bother out of a complicated setup like this.  

### The `miner` Dockerfile 

The `miner` package includes a Dockerfile, which is a plain text file that gives Docker the recipe for setting up an appropriate container. 

This file specifies the following steps that are needed to set up the required environment and run a Spigot Minecraft Server with the RaspberryJuice plug-in: 

- Creates a directory called "minecraft" for the Minecraft server
- Downloads all required files to build a Spigot server (https://www.spigotmc.org) and saves them in the "minecraft" direction
- Builds the Spigot server
- Symlink for the built Spigot server?
- Accepts the End User License Agreement for Minecraft ("eula") (see [here](https://account.mojang.com/documents/minecraft_eula) to see what you are agreeing to with this step)
- Downloads the RaspberryJuice plugin (which we're using for API access) to a subdirectory of the "minecraft" directory called "plugins"
- Install the RaspberryJuice plugin
- Open up the ports required to access the game (port 25565) and the API (4711)
- Start the Minecraft server, [explain options we're using for that]

This Dockerfile is included in the `miner` package. To find it on your computer once you've installed the `miner` package, you can run:


```r
system.file("Dockerfile", package = "miner")
```

This call will return the file pathname on your computer for any the file named "Dockerfile" that come with the `miner` package. 

If you'd like to take a look at the Dockerfile, from R you can run: 


```r
edit(system.file("Dockerfile", package = "miner"))
```

This will open the "Dockerfile" file in the `miner` package in a text editor.

### Building a Docker image

The Dockerfile is a very small plain text file and only gives the recipe for setting up the needed environment and starting a server. To get all the required pieces and be ready to run a container, you need to build a Docker image from this Dockerfile. Once you have installed Docker on your computer (which you can do from [the Docker website](https://www.docker.com)), you open a command line (e.g., the Terminal application on MacOS, on Windows use the Docker Quickstart Terminal), move into the directory with the Dockerfile (using `cd` to change directory), and then build a Docker image based on this Dockerfile by running the following call from a command line:


```bash
docker build -t minecraft .
```

The `docker build` call is the basic call to build a Docker image from a Dockerfile. The option `-t minecraft` tells Docker to give the image the tag "minecraft". By doing this, you can later refer to this image as "minecraft". The `.` at the end of the call tells Docker to build this image based on the file called "Dockerfile" in the current working direction (`.`). 

Once you've built the image, you can check to see that it's in the Docker images on your system by running the following call from a command line: 


```bash
docker images
```

You should see something like this: 

```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
minecraft           latest              2c9e2f2c16d3        3 days ago          1.03 GB
java                latest              d23bdf5b1b1b        4 months ago        643 MB
```

This tells you which Docker images you have on your system, when they were created, how large they are, and the Image ID. If you'd ever like to remove a Docker image from your system, you can do that with the command line call `docker rmi` and the image ID. For example, if you ever wanted to remove the "minecraft" image listed above that you built with the call to `docker build`, you could run: 


```bash
docker rmi 2c9e2f2c16d3
```

### Running a Docker container from the image

Once you have built a Docker image, you can run a container from it. To do that for our Minecraft server, at the command line you should run:


```bash
docker run -ti --rm -p 4711:4711 -p 25565:25565 minecraft 
```

The `docker run` call is the basic call to run a Docker container from a Docker image. The `minecraft` at the end tells Docker which image to run. The `--rm` option cleans up everything from this container after you're done running it. The `-ti` argument runs the call in the interactive terminal mode. The arguments `-p 4711:4711 -p 25565:25565` allow the needed access to the ports for the game itself (port 25565) and the API (4711). After you press Enter you'll see messages about server starting up in your console. Now you can open your regular desktop Minecraft application, select 'Multiplayer', then 'Direct Connect', then enter the IP number for your Docker container. You can find your Docker container IP number by opening another terminal and running `docker-machine IP`. After the Minecraft server has started, the Docker terminal will have a prompt like this `>` where you can enter commands to Minecraft. If you enter `op <player>` and press Enter in the Docker terminal, then you can grant yourself operator status, and you can run game commands such as changing the gamemode (e.g. survival/creative), time, weather, etc. in the Minecraft dekstop app, as usual. If you don't run `op <player>` in the Docker terminal, you will get messages that you don't have permission if you try to run commands in the desktop app. 

## Raspberry Pi

Installing stuff on a Raspberry Pi.
