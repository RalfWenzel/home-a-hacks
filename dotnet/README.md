# .NET hacks
## Target Hardware  Environment
small server i found in company's store room:

````
ralf@beaver1:~$ lspci
00:00.0 Host bridge: Intel Corporation Atom Processor D4xx/D5xx/N4xx/N5xx DMI Bridge (rev 02)
00:1a.0 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #4 (rev 02)
00:1a.1 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #5 (rev 02)
00:1a.2 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #6 (rev 02)
00:1a.7 USB controller: Intel Corporation 82801I (ICH9 Family) USB2 EHCI Controller #2 (rev 02)
00:1c.0 PCI bridge: Intel Corporation 82801I (ICH9 Family) PCI Express Port 1 (rev 02)
00:1c.4 PCI bridge: Intel Corporation 82801I (ICH9 Family) PCI Express Port 5 (rev 02)
00:1c.5 PCI bridge: Intel Corporation 82801I (ICH9 Family) PCI Express Port 6 (rev 02)
00:1d.0 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #1 (rev 02)
00:1d.1 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #2 (rev 02)
00:1d.2 USB controller: Intel Corporation 82801I (ICH9 Family) USB UHCI Controller #3 (rev 02)
00:1d.7 USB controller: Intel Corporation 82801I (ICH9 Family) USB2 EHCI Controller #1 (rev 02)
00:1e.0 PCI bridge: Intel Corporation 82801 PCI Bridge (rev 92)
00:1f.0 ISA bridge: Intel Corporation 82801IR (ICH9R) LPC Interface Controller (rev 02)
00:1f.2 IDE interface: Intel Corporation 82801IR/IO/IH (ICH9R/DO/DH) 4 port SATA Controller [IDE mode] (rev 02)
00:1f.3 SMBus: Intel Corporation 82801I (ICH9 Family) SMBus Controller (rev 02)
00:1f.5 IDE interface: Intel Corporation 82801I (ICH9 Family) 2 port SATA Controller [IDE mode] (rev 02)
02:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
03:00.0 Ethernet controller: Intel Corporation 82574L Gigabit Network Connection
04:04.0 VGA compatible controller: Matrox Electronics Systems Ltd. MGA G200eW WPCM450 (rev 0a)
ralf@beaver1:~$
````
4GB RAM. Put a SSD in there to speed up things a little bit, installed Ubuntu server 18.04 LTS.

## The middleware

First i want to start actively measuring things or passively receive data from all the nice gadgets i have at home.
So basically i need a datastore suited for both timeseries data (measurements) and events.
Also i want see what data i got without the need of writing a GUI app for visualization.

### Elastic Search 6.4 as datastore

Followed standard installation using apt-get. Modified /etc/elasticsearch.yml according to my test requirements (=don't care about security :-)
````
cluster.name: treegarden5
network.host: [ _local_, 192.168.178.10 ]
````
Tested from my local Dev-Laptop (Windows 10, bash on ubuntu on windows or however they call it, Visual Studio 2017 CE)
````
ralf@WS-WER:~/repos/github/home-a-hacks$ curl -X GET "beaver1.fritz.box:9200/_cat/health?v"
````
### Grafana to get things visualized
Followed standard installation using apt-get. Modified nothing.
Tested from my local browser: http://beaver1.fritz.box

### Make them talk to each other.
Grafana's Elastic Search Datasource requires an index name and the name of a field where timestamps are stored.
So i created an index:
````
$ curl -X PUT "beaver1.fritz.box:9200/hah_test?pretty" \
  -H 'Content-Type: application/json' \
  -d'{"mappings":{"_doc": {"properties": {"timestamp": {"type": "date"}}}}}'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "hah_test"
}
````
And put documents in there:
````
$ curl -X PUT "localhost:9200/my_index/_doc/1" -H 'Content-Type: application/json' \
  -d'{ "timestamp": "2018-09-14T20:54:30Z", "Message": "Hello homahacks !" }'
````
I fed the index name and the name of the timestamp field to Elastic Search Datasource in Grafana. Save & Test worked fine.
Created a dashboard, table 


