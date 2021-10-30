 # ==TODO: Create a script for installing all required packages (?)==
# Packages
*Note: packages listed here are arranged from top to bottom in order of installation*
Packages that are **not required** are listed with a prepended \*
## \*Tmux
* Tmux is, as described on its github page:

*A terminal multiplexer. It lets you switch easily between several programs in one terminal, detach them (they keep running in the background) and reattach them to a different terminal.*
* *Note: Tmux is completely optional, not required for the system to function*
* The instructions for installing **on Debian/Ubuntu** via terminal appear below
```bash
sudo apt -y install tmux
```
## Libtrace:
* Libtrace is, as described on its github page:

*A userspace library for processing of network traffic capture from live interfaces or from offline traces*
* Instructions for installing  can be found at https://github.com/LibtraceTeam/libtrace/wiki/Installing-Libtrace
* The instructions for installing **on Debian/Ubuntu** via terminal appear below as of 6/30/2021 @ 5PM
```git
sudo apt-get -y install curl apt-transport-https gnupg
curl -1sLf 'https://dl.cloudsmith.io/public/wand/libwandio/cfg/setup/bash.deb.sh' | sudo -E bash
curl -1sLf 'https://dl.cloudsmith.io/public/wand/libwandder/cfg/setup/bash.deb.sh' | sudo -E bash
curl -1sLf 'https://dl.cloudsmith.io/public/wand/libtrace/cfg/setup/bash.deb.sh' | sudo -E bash
sudo apt-get -y install libtrace4 libtrace4-dev libtrace4-tools
```
## Corsaro3
* Corsaro3 is, as described on its github page:

*A re-implementation of a decent chunk of the original Corsaro which aims to be better suited to processing parallel traffic sources, such as nDAG streams or DPDK pipelines.*
* Instructions for installing can be found at https://github.com/CAIDA/corsaro3
* The instructions for installing **on Debian/Ubuntu** via terminal appear below as of 6/30/2021 @ 5PM
```git
curl https://pkg.caida.org/os/ubuntu/bootstrap.sh | bash
# ^ of course, you should inspect this script first
sudo apt install corsaro
```

## \*mlocate
* mlocate is a tool for finding files based on their names
* The instructions for installing **on Debian/Ubuntu** via terminal appear below
```bash
sudo apt -y intstall mlocate
```
## iPerf3
* iPerf3 is, as described on its website:

*A tool for active measurements of the maximum achievable bandwidth on IP networks. It supports tuning of various parameters related to timing, buffers and protocols (TCP, UDP, SCTP with IPv4 and IPv6). For each test it reports the bandwidth, loss, and other parameters.*
* The instructions for installing **on Debian/Ubuntu** via terminal appear below

```bash
sudo apt -y install iperf3
```
## \*net-tools
- The net-tools package contains several basic networking tools, of which ifconfig is mainly used
- The instructions for installing **on Debian/Ubuntu** via terminal appear below

```bash
sudo apt -y install net-tools
```
## \*xclip
- xclip, as described in its manual:

*Reads from standard in, or from one or more files, and makes the data available as an X selection for pasting into X applications. Prints current X selection to standard out.*
- In simpler terms, it makes putting things into the clipboard for copying and pasting much easier
- The instructions for installing **on Debian/Ubuntu** via terminal appear below

```bash
sudo apt -y install xclip
```

## Docker
- Docker is:
*a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers. Containers are isolated from one another and bundle their own software, libraries and configuration files.*
- For this project, it will be used in the data transfer to UCSD process
- The instructios for installing **on Debian/Ubuntu** via terminal appear below as of 27/7/2021
```bash
# Setup docker repository
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
	
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  
# Install docker engine
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```
The above steps simply install docker on the host machine. The steps to install/configure the "friendly-tagger" that we need for this project are below.
	- They can also be found at https://github.com/CAIDA/telegraf-friendlytagger/
*Note you need go installed prior to this step*
```bash
# First we need to get the relevant things from github
$ git clone https://github.com/CAIDA/telegraf-friendlytagger/

# Before we can build the container, need to make a revision to docker/Dockerfile
# On the first line of the file, change 'FROM golang:alpine' to 'FROM golang:1.15-alpine'
$ docker build -t friendlytag -f docker/Dockerfile .
```
 Now that the container is built, some changes must be made to the telegraf conf file
Open the Docker/telegraf.conf file  with a text editor of your choice and edit the 'OUTPUT PLUGINS' and 'INPUT PLUGINS' to look like the examples below
```docker
###############################################################################
#                            OUTPUT PLUGINS                                   #
###############################################################################


# Configuration for sending metrics to InfluxDB
[[outputs.influxdb]]
    urls = ["http://localhost:8086"]
    database = "testdb"
    username = "writer"
    password = "password123"
    tagexclude = ["drop", "host", "debug", "meas"]
    [outputs.influxdb.tagdrop]
      debug = ["true"]

[snip]

###############################################################################
#                            INPUT PLUGINS                                    #
###############################################################################
[[inputs.tail]]

files = ["/home/test/testing/corsarotrace_output/*"]

## Read file from beginning.
# from_beginning = false

## Whether file is a named pipe
# pipe = false

## Method used to watch for file updates.  Can be either "inotify" or "poll".
# watch_method = "inotify"

## Maximum lines of the file to process that have not yet be written by the
## output.  For best throughput set based on the number of metrics on each
## line and the size of the output's metric_batch_size.
# max_undelivered_lines = 1000

## Character encoding to use when interpreting the file contents.  Invalid
## characters are replaced using the unicode replacement character.  When set
## to the empty string the data is not decoded to text.
##   ex: character_encoding = "utf-8"
##       character_encoding = "utf-16le"
##       character_encoding = "utf-16be"
##       character_encoding = ""
# character_encoding = ""

## Data format to consume.
## Each data format has its own unique set of configuration options, read
## more about them here:
## https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md
data_format = "influx"

## Set the tag that will contain the path of the tailed file. If you don't want this tag, set it to an empty string.
# path_tag = "path"

## multiline parser/codec
## https://www.elastic.co/guide/en/logstash/2.4/plugins-filters-multiline.html
#[inputs.tail.multiline]
## The pattern should be a regexp which matches what you believe to be an indicator that the field is part of an event consisting of multiple lines of log data.
#pattern = "^\s"

## The field's value must be previous or next and indicates the relation to the
## multi-line event.
#match_which_line = "previous"

## The invert_match can be true or false (defaults to false). 
## If true, a message not matching the pattern will constitute a match of the multiline filter and the what will be applied. (vice-versa is also true)
#invert_match = false

##After the specified timeout, this plugin sends the multiline event even if no new pattern is found to start a new event. The default is 5s.
#timeout = 5s
```
Now that the input and output have been configured, the container is ready to run. 
```bash
# Note: this command should be ran from the same directory as before, or the topmost directory of the git clone command
$ docker run --name <choose a name> -v /home/honeynet/test_trace:/tmp:ro -v $(pwd)/docker/telegraf.conf:/etc/telegraf/telegraf.conf:ro friendlytag telegraf
# Now you should be seeing tagger output being printed to the console 
```

## go
- go is a programming language that is required by the "friendly-tagger" from Shane that we will be using for this project
- The instructions for installing go **on Debian/Ubuntu** via terminal appear below.
```bash
sudo snap install go --classic
```
If snap isn't installed on your system, instructions for installing it can be found @ https://snapcraft.io/docs/installing-snap-on-ubuntu

## InfluxDB
InfluxDB is an open-source time series database. For this project it will be used with the friendly-tagger container for transferring data to UCSD
- The instructions for installing InfluxDB **on Debian/Ubuntu** via terminal appear below as of 28/7/2021
```bash
# Note these commands must be ran as the super user
## get dependencies figured out
$ wget -qO- https://repos.influxdata.com/influxdb.key | gpg --dearmor > /etc/apt/trusted.gpg.d/influxdb.gpg
$ export DISTRIB_ID=$(lsb_release -si); export DISTRIB_CODENAME=$(lsb_release -sc)
$ echo "deb [signed-by=/etc/apt/trusted.gpg.d/influxdb.gpg] https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" > /etc/apt/sources.list.d/influxdb.list
# install influx
$ sudo apt-get update && sudo apt-get install influxdb
# Then start the db with 
$ sudo service influxdb start
```
For this system to function properly, there are more hoops to jump through. Things that need to be created:
- Database for writing our data to/UCSD querying
- Admin user so that influx can respond to HTTP requests
- A user for UCSD to read data
- A user for the container to write data

Most of these steps are fairly simple and can be completed quickly. Useful documentation for working with influx can be found @ https://docs.influxdata.com/influxdb/v1.8/
- Creating database:
```SQL
# First start the CLI
$ influx -precision rfc3339 
Connected to http://localhost:8086 version 1.8.6
InfluxDB shell version: 1.8.6
> CREATE DATABASE testdb
> 
```
*Note, a new prompt appearing after hitting enter is the expected behavior. This is how most commands show they were successful in influx*
- Creating admin user and users for reading and writing data
```SQL
> CREATE USER administrator WITH PASSWORD 'password' WITH ALL PRIVILEGES
> SHOW users
user          admin
----          -----
administrator true
>
> CREATE USER "reader" WITH PASSWORD 'password123'
> GRANT READ ON "testdb" TO "reader"
> CREATE USER "writer" WITH PASSWORD 'password456'
> GRANT WRITE ON "testdb" TO "writer"
> SHOW users
user          admin
----          -----
administrator true
reader        false
writer        false
> 
```

## build-essential
build-essential is a package that is used in compiling debian packages. It includes things like 'make' and g++ normally, so install this before manually installing g++
- instructions for installing **on Debian/Ubuntu** can be found below
```bash
sudo apt -y install build-essential
```
## g++
g++ is a C++ compiler that is needed by harpoon
- instructions for installing **on Debian/Ubuntu** can be found below
```bash
sudo apt -y install g++
```

## eXpat library
eXpat library is a C library for paring yaml. Like g++, it is needed by harpoon
- instructions for installing **on Debian/Ubuntu** can be found below
```bash
sudo apt -y install expat
sudo apt -y install libexpat1-dev
```

## harpoon
harpoon is a tool for generating flow-level traffic. In simpler terms, it is a tool that will test how much traffic our system can handle before a breakdown in performance.
*Note this software was provided by Prof. Barford through his personal cs page. He asked us not to distribute the code so I will refrain from putting the download location in this document*
- instructions for installing **on Debian/Ubuntu** can be found below
- Note: an INSTALL file is included in the archive I received from Prof. Barford. The below instructions are derived from this file.
```bash
# First, the requirements (found in INSTALL) must be installed
# The required packages for harpoon are:
	# c++ compiler (we used g++)
	# eXpat library
	# POSIX threds
# POSIX threads came pre-installed on the test system, so there are no instructions for installing it in this doc
# Instructions for installing the other required packages are located elsewhere in this document

# Note: all instructions from this point forward will assume you are in the top level directory
$ ./configure
# This script will tell you if you are missing anything that is required
$ make
# This makefile failed for me a few times. Each time it failed, it would say something about a file not having a file/library included
# To fix this, locate the file with the missing include and manually add it. Then, go back to the top level directory and run 'make' again
$ sudo make install
```
