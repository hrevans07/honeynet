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