# Arris CM8200 to InfluxDB

This is a python script to webscrape the Arris CM8200 web interface and place data into Influxdb for graphing in Grafana. Its intended use is on the NBN MTM HFC network, specifically the CM8200B. It will however work with other arris modems that use the same UI and others with some modification.

This assumes that Grafana and Influxdb are already installed and working and that you have a basic understanding of both.
This also asusmes that the Arris modem is accessible from within the end users LAN(with or without NAT rules etc).

Just a note, if there is a flap or dropout the Arris NTD will do a soft reboot thus blocking access to the WebUI requiring a factory reset using the reset pin for ~5seconds. Unfortuantly this is just how it is from the NBN overlords.

As an UBNT user the below should gain you access assuming NBN has not disabled access. Has been tested on Edgerouter and USG products.
```bash
risbo@Edge:~$ configure
[edit]
risbo@Edge#

set interfaces pseudo-ethernet peth0 address 192.168.0.2/24
set interfaces pseudo-ethernet peth0 description 'Modem Access'
set interfaces pseudo-ethernet peth0 link eth0
commit
set service nat rule 5000 description 'masquerade for HFC Modem'
set service nat rule 5000 outbound-interface peth0
set service nat rule 5000 type masquerade
commit
save
```
![Grafana Overview](https://raw.githubusercontent.com/risb0r/Arris-CM8200-to-InfluxDB/master/images/overview.png)

## Installation

Clone to machine. I chose to personally run this from the `/opt/` directory.
```bash
git clone https://github.com/risb0r/Arris-CM8200-to-InfluxDB.git arris_stats
```

Install python3, python3pip and python3-lxml as required.
```bash
sudo apt install python3 python3-pip python3-lxml
```
Use the pip package manager pip3 to install necessary requirements.
Note: Requirements may also have to be installed as `root` depending on your setup.
```bash
pip3 install -r requirements.txt
```

Setup influx with a database
```bash
$ influx
> CREATE DATABASE cm8200b_stats
```
Ensure that the database was created
```bash
> show databases
name: databases
name
----
cm8200b_stats <------
```

Adjust cm8200_stats.py to suite your requirements
```python
# Change settings below to your influxdb - database needs to be created or existing db
# creates 5 tables - downlink, uplink, fw_ver, uptime, event_log

influxip = "127.0.0.1"
influxport = "8086"
influxdb = "cm8200b_stats"
influxid = "admin"
influxpass = ""

# cm8200b URLs - leave these as is unless a firmware upgrade changes them

linestats = "http://192.168.0.1/cmconnectionstatus.html"
generalstats = "http://192.168.0.1/cmswinfo.html"
logstats = "http://192.168.0.1/cmeventlog.html"
```

## Usage
### Running the webscraper

Standalone (once off)
```bash
/usr/bin/python3 /opt/arris_stats/cm8200b_stats.py
```

![cm8200_stats.py Output](https://raw.githubusercontent.com/risb0r/Arris-CM8200-to-InfluxDB/master/images/output.png)

As cron
```bash
sudo crontab -e
```
Place the below into crontab. Ctrl + X to exit.
```bash
# m h  dom mon dow   command

*/5 * * * * /usr/bin/python3 /opt/arris_stats/cm8200b_stats.py
```

### Setting up Grafana

Setup the data source as below

![Datasource Overview](https://raw.githubusercontent.com/risb0r/Arris-CM8200-to-InfluxDB/master/images/datasource.png)


Import the .json

If the images are out of wack check the grafana.ini file for the following config change.
```bash
$ sudo nano /etc/grafana/grafana.ini

[panels]
# If set to true Grafana will allow script tags in text panels. Not recommended as it enable XSS vulnerabilities.
disable_sanitize_html = true
```
## To Do List        

Auto scrape [Whirlpool](https://whirlpool.net.au/wiki/cmts-upgrades) and plug in the CMTS info from the wiki rather than just filling out a text box. Personally, I can't be bothered or care too much for something that will go mostly unchanged.

# Thanks
Thanks to [Andy Fraley](https://github.com/andrewfraley/arris_cable_modem_stats) for the initial starting point and grafana json.
Thanks to Luckst0r for the current python base code and doing some testing.
There are a few of us lurking on the [AussieBroadband Unofficial Discord](https://forums.whirlpool.net.au/archive/2713195) if any discussion is desired based on this.

Thanks to the team at [Aussie Broadband](https://www.aussiebroadband.com.au/) for providing dope internet on a government cockup.

CMTS info is a static item and is available on [Whirlpool](https://whirlpool.net.au/wiki/cmts-upgrades) thanks to [Roger Ramband](https://forums.whirlpool.net.au/user/117375) for making this data readily available. This data is only relevant to those connected via the NBN in Australia. Can be skipped, removed, deleted, whatever for others.
