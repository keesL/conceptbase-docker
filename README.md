# Running ConceptBase in Docker

[ConceptBase.cc](http://conceptbase.sourceforge.net) is a powerful tool for metamodeling and engineering of customized modeling languages, based on a multi-user deductive database system. 

The client/server system has clients written in Java, but the server requires a Linux installation. While not difficult to install, not everybody can (or wants) to install and operate a virtual machine on the Mac OS X or Windows system. Fortunately, ConceptBase runs without any issue in (Docker)[Docker].

The basic workflow is as follows:

1. Extract the installation files from the installer

2. Build a docker image

3. Create a docker container

4. Run the container as often as you want



## Conceptbase Server

### Prepare installation files

This is maybe the trickiest part of the entire installation process, since it requires the availability of an XWindows installation. Since the Conceptbase installer is a GUI application, it needs to display somewhere. The server has no GUI components, so once we are done extracting the files, we don't need it anymore.

1. On Mac OS X, install [XQuartz](https://www.xquartz.org). Windows has XMing, but I have not tested it.

2. Run the XServer. From within an xterm, execute the command `xhost + 127.0.0.1`. This allows clients on localhost (i.e., your computer) to display GUIs on your display.

3. Create a folder/directory in which you want to keep your ConceptBase files. I'll assume you call it `cb`.

3. Download the ConceptBase.cc [installer](http://conceptbase.sourceforge.net/CB-Download.html) and copy it into the `cb` directory.

4. Create a temporary docker container that maps your `cb` directory into `/cb` on the image, and then launch a `bash` shell in it.

To do run, run the following command from a command-line interface on your Mac:

    docker run -ti -v `pwd`/cb:/cb -e DISPLAY=host.docker.internal:0 debian /bin/bash

5. Follow the instructions for Ubuntu 20.04, listed on the page with (installation instructions)[http://conceptbase.sourceforge.net/CB-Download.html]
fix dependencies in the image and run CBinstaller. 

Tell the installer to set the installation directory to be `/cb`.

6. Disconnect. At this point, running `ls cb` should list the extracted ConceptBase files.

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

    docker create --name cbserver 
    	--publish 4001:4001 \
		--mount type=bind,src=`pwd`/cbdata,dst=/cb/cbdata \
		cbserver:latest

If you choose to change the port number or the location of the cbdata directory, you must update the command listed above.

The container needs to be created only once. After, that, you can run it as often as needed by issuing the command

    docker start --attach cbserver


NOTE: the server is set to shut down after the last client disconnects.

## Running the client

The client will use XWindows remote display to show its GUI. To do so, must
specify the DISPLAY environment and have local X server running with `xauth +
127.0.0.1`

First we create the container

    docker create --name cbclient 
    	--env DISPLAY=host.docker.internal:0 \
		--mount type=bind,src=`pwd`/cbexport,dst=/cb/cbexport \
		--env CB_SERVER=host.docker.internal cbclient:latest

And then we can run it as often as we want

    docker start cbclient

If nothing happens after issuing this command, make sure that XWindows is running and that `xhost` access controls are set appropriately.

## Running all-in-one

I also built an all-in-one experiment that runs the client and the server in one go.	
    docker-compose up
