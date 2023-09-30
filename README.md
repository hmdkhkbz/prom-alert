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

sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana -y
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
