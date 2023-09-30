....This is a Step-by-Step guide for installing prometheus, alertmanager, grafana and nginx on ubuntu 22.04 (native/baremetal) 

# first for bypass 403 error in iran, it is recommanded for set SNI proxy on your server.
 
rm /etc/resolv.conf
echo "nameserver 10.202.10.202 " | sudo tee -a /etc/resolv.conf
echo "nameserver 10.202.10.102 " | sudo tee -a /etc/resolv.conf

apt update
apt install -y prometheus

# it is recommanded for checking the content of "/etc/prometheus/prometheus.yml" and customize for your environment.

systemctl start prometheus
systemctl enable prometheus

# if you need to authenticate nginx incoming run "apt install -y apache2-utils"  and do "htpasswd -c /etc/nginx/.htpasswd hkh"

apt install -y nginx

# it is recommanded for edit " /etc/nginx/nginx.conf " and costomize cinfiguration for your environment.

systemctl start nginx
systemctl enable nginx

# we install node exporter with binary

wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz 

tar xvfz alertmanager-0.25.0.linux-amd64.tar.gz

mkdir -p /etc/alertmanager

mkdir -p /var/lib/alertmanager

mkdir -p /etc/amtool

cp alertmanager-0.25.0.linux-amd64/alertmanager /usr/local/bin/

cp alertmanager-0.25.0.linux-amd64/alertmanager.yml /etc/alertmanager

cp alertmanager-0.25.0.linux-amd64/amtool /usr/local/bin/

sudo useradd -M -r -s /bin/false alertmanager

chown alertmanager:alertmanager /usr/local/bin/alertmanager

chown -R alertmanager:alertmanager /etc/alertmanager

chown alertmanager:alertmanager /var/lib/alertmanager


# it is recommanded for checking the content of "/etc/systemd/system/alertmanager.service" and customize for your environment.

sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter

# for alerting there is alert rules in alerts.yml 

sudo apt-get install -y software-properties-common

sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

sudo apt-get update

sudo apt-get install grafana -y

sudo systemctl start grafana-server

sudo systemctl enable grafana-server

# know we need to configure iptables with following rules (Hardening)

iptables -A INPUT -p tcp --dport 9090 -j DROP

iptables -A INPUT -p tcp --dport 3000 -j DROP

iptables -A INPUT -p tcp --dport 9093 -j DROP

iptables -A INPUT -p tcp --dport 9100 -j DROP

iptables -A INPUT -p tcp --dport 80 -j DROP

iptables -I INPUT 1 -s 127.0.0.1 -j ACCEPT

sudo apt-get install iptables-persistent

sudo netfilter-persistent save

# for generate TLS/SSL Certificate we are using certbot as below:



sudo apt update

sudo apt install certbot python3-certbot-nginx -y

sudo systemctl stop nginx

sudo certbot --nginx -d grafana.hmdkhkbz.ir

sudo systemctl start nginx


# for monitor another nodes you need to configure node exporter on that node


wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz

tar xvfz node_exporter-1.2.2.linux-amd64.tar.gz

mv node_exporter-1.2.2.linux-amd64/node_exporter /usr/local/bin

# it is recommanded to configure "/etc/systemd/system/node_exporter.service" and customize for your environment.

useradd -rs /bin/false node_exporter

systemctl daemon-reload

systemctl start node_exporter

systemctl enable node_exporter

systemctl status node_exporter
