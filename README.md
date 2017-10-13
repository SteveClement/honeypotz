# honeypotz
## Installing ELK Server
Just follow:
* https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-ubuntu-14-04
### Add GeoIP database to logstash
```
cd /etc/logstash
sudo curl -O http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
sudo gunzip GeoLiteCity.dat.gz
```

### Add cowrie logs configuration to logstash
Copy 11-cowrie-filter.conf to /etc/logstash/conf.d.
Then run:
```
sudo service logstash restart
```
## Deploying Cowrie Honeypot
### Changing ssh port
```
sudo vi /etc/ssh/sshd_config
# What ports, IPs and protocols we listen for
Port 7890
/etc/init.d/ssh restart
```
Save and exit.
### Autorestart kibana
Due to memory limitations in the droplet (2 gb), kibana eventually crashes, you can set it to autorun using supervisor.
```
apt-get install supervisor
```
Add this file to /etc/supervisor/conf.d, then:
```
service supervisor restart
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
