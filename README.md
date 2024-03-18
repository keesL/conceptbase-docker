# Running ConceptBase in Docker

[ConceptBase.cc](http://conceptbase.sourceforge.net) is a powerful tool for metamodeling and engineering of customized modeling languages, based on a multi-user deductive database system. 

The client/server system has clients written in Java, but the server requires a Linux installation. While not difficult to install, not everybody can (or wants) to install and operate a virtual machine on the Mac OS X or Windows system. Fortunately, ConceptBase runs without any issue in (Docker)[Docker].

The basic workflow is as follows:

1. Build the Docker images

2. Create container(s) for the server (and perhaps for the client as well)

3. Run the container as often as you want



## Conceptbase Server

### Prepare installation files

1. Download the ConceptBase.cc [installer](http://conceptbase.sourceforge.net/CB-Download.html) (CBinstaller.jar) and copy it into the `cb` directory.


## Building the images

The `conceptbase` image is the foundation on which the other two are built. It includes all the files needed to run either a client or a server.

Step 1: Build the base image

    docker build --tag conceptbase:latest -f Dockerfile-base .

Step 2: Build the server image

    docker build --tag cbserver:latest -f Dockerfile-server .

Step 3: Build the client image

    docker build --tag cbclient:latest -f Dockerfile-client .


Make sure to issue these commands in the parent directory of the `cb` folder that you created earlier.

Note: the client image is not strictly needed, since the clients are all implemented in Java. Still; I find it easier to just run from the same code base.

The files will be included into the `/cb/` directory on the image, which is owned by the `cb` user

## Running the server

The server will listen on a TCP port, and can write its database persistently to disk. To store the database in a directory of the host, mount a volume in its place.

Supported environment variables

* `CB_PORT` (default: 4001) TCP port on which the server will listen
* `CB_DATA` (default: /cb/cbdata) Directory containing db files
* `CB_MODE` (default: persistent) Alternative is to use nonpersistent
* `CB_TRACE` (default: low) Tracing level used by the server. 

Create the server container

    docker create --name cbserver \
    	--publish 4001:4001 \
		--mount type=bind,src=`pwd`/cbdata,dst=/cb/cbdata \
		cbserver:latest

If you choose to change the port number or the location of the cbdata directory, you must update the command listed above.

The container needs to be created only once. After, that, you can run it as often as needed by issuing the command

    docker start --attach cbserver


NOTE: the server is set to shut down after the last client disconnects.

## Running the client

NOTE: The client GUIs will run natively in your OS. You do NOT need to run them inside Docker. However, if you choose to do it anyway, follow these instructions.

The client will use XWindows' remote display features to show its GUI. If you choose to run the client from within a container, you have an XWindows system running on your computer (e.g., XQuartz on a Mac, or XMing on a Windows PC). Make sure that your Xserver accepts incoming connections from localhost (`xauth + 127.0.0.1`).


First we create the container

    docker create --name cbclient \
    	--env DISPLAY=host.docker.internal:0 \
		--mount type=bind,src=`pwd`/cbexport,dst=/cb/cbexport \
		--env CB_SERVER=host.docker.internal cbclient:latest

And then we can run it as often as we want

    docker run -e CB_HOME=/cb -ti cbclient bin/CBjavaInterface

If nothing happens after issuing this command, make sure that XWindows is running and that `xhost` access controls are set appropriately.

## Running all-in-one

I also built an all-in-one experiment that runs the client and the server in one go.	
    docker-compose up


