# honeypotz
![Kibana Dashboard](https://github.com/xluccianox/honeypotz/blob/master/kibana-dashboard.png)
**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [honeypotz]
	- [Installing ELK Server](#installing-elk)
		- [Add GeoIP database to logstash](#add-kibana-geopip)
		- [Add cowrie logs configuration to logstash](#)
	- [Deploying Cowrie Honeypot](#)
		- [Changing ssh port](#)
		- [Autorestart elasticsearch & kibana](#)
		- [Docker](#)
		- [Logging to ELK Server](#)
			- [Copy the CA Cert from ELK Server to the Honeypot machine](#)
			- [Installing & Configuring Filebeats](#)
		- [Check connection to the ELK Server](#)
		- [Check for cowrie logs in ELK Server](#)
	- [References and guides used](#)
    
## [Installing ELK Server](#installing-elk)
Just follow:
* https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-ubuntu-14-04
### [Add GeoIP database to logstash](#add-kibana-geopip)
```
cd /etc/logstash
sudo curl -O http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
sudo gunzip GeoLiteCity.dat.gz
```

### Add cowrie logs configuration to logstash
Copy [https://github.com/xluccianox/honeypotz/blob/master/11-cowrie-filter.conf 11-cowrie-filter.conf] to /etc/logstash/conf.d.
Then run:
```
sudo service logstash restart
```
## Deploying Cowrie Honeypot
* Download https://github.com/micheloosterhof/docker-cowrie/blob/master/Dockerfile.
* In the Dockerfile location, run:
```
sudo apt-get install docker-io
docker build -t cowrie .
```
Create this directory structure in /.
/cowrie/
├── docker-cowrie

│   ├── Dockerfile

├── etc

│   └── cowrie.cfg

└── log
    └── tty
    
Then:

```
chown -R 999:1000 /cowrie/log 
docker run -d -p 22:2222 -v /cowrie/etc/cowrie.cfg:/cowrie/cowrie-git/etc/cowrie.cfg -v /cowrie/log:/cowrie/cowrie-git/log cowrie
```

### Changing ssh port
```
sudo vi /etc/ssh/sshd_config
# What ports, IPs and protocols we listen for
Port 7890
/etc/init.d/ssh restart
```
Save and exit.
### Autorestart elasticsearch & kibana
Due to memory limitations in the droplet (2 gb), kibana or elasticsearch eventually crashes, you can set it to autorun using supervisor.
```
sudo apt-get install supervisor
```
* Add [https://github.com/xluccianox/honeypotz/blob/master/kibana.conf this file] to /etc/supervisor/conf.d
```
ln -s /etc/elasticsearch/ /usr/share/elasticsearch/config
chown -R elasticsearch:elasticsearch /etc/usr/share/elasticsearch
chown -R elasticsearch:elasticsearch /var/log/elasticsearch
```
* Add [https://github.com/xluccianox/honeypotz/blob/master/elasticsearch.conf this file] to /etc/supervisor/conf.d
Then:
```
supervisorctl reread
supervisorctl update
sudo service supervisor restart
```
To list service statusses:
```
> sudo supervisorctl status
elasticsearch                    RUNNING    pid 22811, uptime 0:00:01
kibana                           RUNNING    pid 22812, uptime 0:00:01
```

### Docker
### Logging to ELK Server
#### Copy the CA Cert from ELK Server to the Honeypot machine
```
scp -P 7890 /etc/pki/tls/certs/logstash-forwarder.crt root@<COWRIE_HOST>:/tmp
```
#### Installing & Configuring Filebeats
Follow:
* https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-ubuntu-14-04#set-up-filebeat-(add-client-servers)
Using this filebeats.yml configuration.
### Check connection to the ELK Server
```
curl -v --cacert /etc/pki/tls/certs/logstash-forwarder.crt https://<ELK_SERVER_HOST>:5044
```
### Check for cowrie logs in ELK Server
Run from a terminal inside ELK Server
```
curl 'http://localhost:9200/_search?q=cowrie&size=5
```

## References and guides used
* https://github.com/micheloosterhof/cowrie
* https://www.digitalocean.com/community/tutorials/how-to-set-up-and-use-digitalocean-private-networking
* https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-ubuntu-14-04#install-nginx
* https://www.digitalocean.com/community/tutorials/how-to-map-user-location-with-geoip-and-elk-elasticsearch-logstash-and-kibana
* https://github.com/micheloosterhof/cowrie/tree/master/doc/elk
* https://www.digitalocean.com/community/tutorials/how-to-install-and-manage-supervisor-on-ubuntu-and-debian-vps
