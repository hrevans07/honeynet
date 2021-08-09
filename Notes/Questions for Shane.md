## 27/7/2021
### Questions about corsaro
- The first solution (using sysctl to change kernel params) seemed to work at first, but eventually there was the "unable to push tagged..." error
	- I changed the setting to match those that you sent me 
- Error at bottom of tracertstats when it terminates
```bash
ndag:lo,225.200.0.100,8999(lo,225.200.0.100,8999): Walked past the end of the nDAG receive buffer, probably due to a invalid taghdr->pktlen, in ndag_prepare_packet_stream_corsarotag()
```
- tracertstats doesn't die when corsarotagger gives the "unable to push packets ..." error
- When running corsarotrace, get warning about threads observing packet drops
	- No correlation with the "unable to push packets..." error
		- There are instances where the error occurs and there are no dropped packets as well as instances where it doesn't occur and still no dropped packets
	- In fact, I haven't been able to see any dropped packets running tracertstats on the tagger multicast
```bash
[09:50:12.462] corsarotrace: warning: worker thread 0 has observed 8 packets dropped in the past interval (8 instances) -- 430
[09:50:12.462] corsarotrace: worker thread 0 has observed 8 packets from previous interval during interval 0
```
- When I change the mtu value for tracemcast to less than the value in the tagger config file, tracertstats starts reporting missing packets.
	- When I keep the mtu value the same as the value in the tagger config, no missing packets reported
- If the only point of tracemcast is to create a multicast for corsarotagger to read from, why not just have corsarotagger read from the interface itself? In the example config file on github (https://github.com/CAIDA/corsaro3/blob/master/docs/corsarotagger-README.md), in the inputuri option it says it could be an interface
### Questions about friendly-tagger
- While trying to build the docker image for the friendly tagger, the docker/Dockerfile script calls a makefile that doesn't seem to exist.
```go
RUN cd /go/src/github.com/influxdata/telegraf/ && make && go install -ldflags "-w -s" ./cmd/telegraf
```

## 2/8/2021
```
What happens if you just run `tracertstats` connected directly to the multicast group used by tracemcast? Does that happily run fine for a long period of time without warnings or errors?
```
- Yes, it runs perfectly fine for long periods of time with this configuration; I couldn't get it to produce any errors/warnings and there were no missing/dropped packets.
```
I'm starting to wonder if the two multicasters are interfering with each other somehow -- is there a spare network interface on your machine that you could have corsarotagger use to emit multicast traffic instead? Or maybe even try configuring a virtual interface and multicast to that instead?
```
- There are no spare network interfaces, at least on this testing VM.
- I added virtual interface with instructions found @ https://unix.stackexchange.com/questions/152331/how-can-i-create-a-virtual-ethernet-interface-on-a-machine-without-a-physical-ad
- Still get the same "unable to push tagged packets.." error with this new configuration
- Included screenshots for debugging of: 
	- new ifconfig
	- new corsarotagger config
	- tracertstat comand is now: ``` sudo tracertstats -i 1 ndag:"eth20:0",225.200.0.100,8999```