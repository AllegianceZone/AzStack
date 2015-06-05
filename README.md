Backups:
```
  Trac: 
    sudo tar -czvf /home/imago/dist-packages.tar.gz /usr/local/lib/python2.7/dist-packages
    sudo -u postgres pg_dump trac > /home/imago/trac.sql
  Discourse: 
    login as admin to website and use the backup gui
  AZ CGI:
```

Swap:
```
  sudo install -o root -g root -m 0600 /dev/null /swapfile
  sudo dd if=/dev/zero of=/swapfile bs=1k count=1024k
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo "/swapfile       swap    swap    auto      0       0" | sudo tee -a /etc/fstab
  sudo sysctl -w vm.swappiness=10
  echo vm.swappiness = 10 | sudo tee -a /etc/sysctl.conf
```
Discourse:
```
  sudo wget -qO- https://get.docker.com/ | sh
  sudo usermod -aG docker Imago
  sudo mkdir /var/discourse
  sudo git clone https://github.com/discourse/discourse_docker.git /var/discourse
  cd /var/discourse
  sudo cp samples/standalone.yml containers/app.yml
  sudo nano containers/app.yml
    see app.yml in this repo
  sudo ./launcher bootstrap app
  sudo ./launcher start app
    login to website as admin and enable restore, restore latest bacukup in this repo via gui
    reboot
```
Trac:
```
  cd /var/discourse
  sudo ./launcher enter app
  sudo apt-get install python-dev
  wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python
  easy_install psycopg2
  easy_install Trac
  sudo -u postgres createdb trac
  sudo -u postgres psql template1 -c "CREATE USER tracuser WITH PASSWORD 'allegdb'; GRANT ALL PRIVILEGES ON DATABASE trac to tracuser;" 
  sudo pico /etc/postgresql/9.3/main/pg_hba.conf
    change:
      local all   all   peer
    to:
      local all   all   trust
  sudo sv restart postgres
  sudo trac-admin /opt/trac initenv
    use connection string:
      postgres://tracuser:allegdb@/trac
  sudo -u postgres psql trac -f trac.sql
    get trac.sql from backup
  cd /etc/service
  sudo mkdir tracd
  cd tracd
  sudo nano run
    #!/bin/sh
    exec 2>&1
    OAUTHLIB_INSECURE_TRANSPORT=1 PYTHON_EGG_CACHE=/var/www/.python-eggs exec chpst -u www-data -U www-data /usr/local/bin/tracd -s -auth="*,/opt/trac/.htpasswd,/opt/trac" --port 8000 /opt/trac
  chmod +x run
  cd /opt/trac
  sudo nano .htpasswd
    Imago:$apr1$n4fw6CCM$qTdtelsxSqA4japDs5U6m0
    builder:$apr1$.SjXPbsw$lDmxsS/983hPrNst1SqkL/
  chown www-data:www-data .htpasswd
  cd /var/www
  mkdir .python-eggs
  chown www-data:www-data .python-eggs
  tar -xzf /home/imago/dist-packages.tar.gz /usr/local/lib/python2.7
  put trac.conf in /etc/nginx/conf.d
  sv start tracd
```
AZ CGI:
```
sudo /etc/nginx/conf.d/cgi.conf


server {
root  /var/www;
        server_name azforum.cloudapp.net ;
    listen              80;
    listen              443 ssl;
    ssl_certificate     azforum.cloudapp.net.crt;
    ssl_certificate_key azforum.cloudapp.net.key;

  location / {
      index  index.html index.cgi;
  }

  location ~ \.pl|cgi$ {
      try_files $uri =404;
      gzip off;
      fastcgi_pass  127.0.0.1:8999;
      fastcgi_index index.cgi;
      fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
      include fastcgi_params;
      }
}

sudo nano /etc/nginx/azforum.cloudapp.net.crt
-----BEGIN CERTIFICATE-----
MIICaTCCAdICCQDf3L2xBJW/+TANBgkqhkiG9w0BAQsFADB5MQswCQYDVQQGEwJV
UzETMBEGA1UECAwKV2FzaGluZ3RvbjEQMA4GA1UEBwwHUmVkbW9uZDEkMCIGA1UE
CgwbQWxsZWdpYW5jZSBab25lIERldmVsb3BtZW50MR0wGwYDVQQDDBRhemZvcnVt
LmNsb3VkYXBwLm5ldDAeFw0xNDA5MjQxNjAwMTZaFw0xNzA5MjMxNjAwMTZaMHkx
CzAJBgNVBAYTAlVTMRMwEQYDVQQIDApXYXNoaW5ndG9uMRAwDgYDVQQHDAdSZWRt
b25kMSQwIgYDVQQKDBtBbGxlZ2lhbmNlIFpvbmUgRGV2ZWxvcG1lbnQxHTAbBgNV
BAMMFGF6Zm9ydW0uY2xvdWRhcHAubmV0MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCB
iQKBgQCzOgStxMYXN3gNSMNTwa0tZWyTUgKOCDDTx9h1LVxvqMwQITVyhw4xpPWL
I3XSxBBqRMz/Sv8LucCmyS5zp6mMn4sxddmSudAYQNsFJmmqp0E9uuNI+L2rzG39
xER5QvB7deDtBc32anzGbCZAprR74IcRtubCMMtBpS/5El1wewIDAQABMA0GCSqG
SIb3DQEBCwUAA4GBAEf/qlf7SikbNWIcTSxAh1kmaSUKpGlRDx7BuBfeFTiWqWDs
EB+Lf3QOw+Fj1IgDptL8e12PI8Jwrd2qwigYpsWcGk+rcU02aRVZSyjahMQbdOXc
KO0R4As6fMlwK3/AdFYTs7CC9NoJR7UG9meautMXMwt/L6vD15Vt2aLBgZum
-----END CERTIFICATE-----

sudo nano /etc/nginx/azforum.cloudapp.net.csr
-----BEGIN CERTIFICATE REQUEST-----
MIIBuTCCASICAQAweTELMAkGA1UEBhMCVVMxEzARBgNVBAgMCldhc2hpbmd0b24x
EDAOBgNVBAcMB1JlZG1vbmQxJDAiBgNVBAoMG0FsbGVnaWFuY2UgWm9uZSBEZXZl
bG9wbWVudDEdMBsGA1UEAwwUYXpmb3J1bS5jbG91ZGFwcC5uZXQwgZ8wDQYJKoZI
hvcNAQEBBQADgY0AMIGJAoGBALM6BK3Exhc3eA1Iw1PBrS1lbJNSAo4IMNPH2HUt
XG+ozBAhNXKHDjGk9YsjddLEEGpEzP9K/wu5wKbJLnOnqYyfizF12ZK50BhA2wUm
aaqnQT2640j4vavMbf3ERHlC8Ht14O0FzfZqfMZsJkCmtHvghxG25sIwy0GlL/kS
XXB7AgMBAAGgADANBgkqhkiG9w0BAQsFAAOBgQCtOlw/TRsBoRwbty2jmYluYe0Z
okCS/uGeY2iGq6q0MZc471Yej0JEeuhO8jM4kCyM2M39Odv54g1eod6LoGqh3w98
2SeGJD5mDUERF4QqIlUjT/zUh6e+cFjSzYTtzHS5bS8+YnIfqS+Wf5tbYAcgP3yj
5mb1r+JPyknbZTD2GQ==
-----END CERTIFICATE REQUEST-----

sudo nano /etc/nginx/azforum.cloudapp.net.key

-----BEGIN RSA PRIVATE KEY-----
MIICXgIBAAKBgQCzOgStxMYXN3gNSMNTwa0tZWyTUgKOCDDTx9h1LVxvqMwQITVy
hw4xpPWLI3XSxBBqRMz/Sv8LucCmyS5zp6mMn4sxddmSudAYQNsFJmmqp0E9uuNI
+L2rzG39xER5QvB7deDtBc32anzGbCZAprR74IcRtubCMMtBpS/5El1wewIDAQAB
AoGAMxOkw7ThUzp+nyKOb+8xIE/YSn/DtKCG8cPxXyuHPVcYmLwuFC6DEAjX5Ug8
ys0PdImY9mR0HO5aBe4tq84rViqKhKOtYDsIQaP0iC0/CEW5ldtiMJxNJ4mjkyax
KpN9q4x2Emnr0nm+ILeMs5IISYNExJzO1k12BNBV/jsEZOECQQDqK2BpimMtZQIj
9r84zomEOEAkbHDKFaa3LilYYkr9FRo3ooAR1qi+nHmsxDjLE9IxlG8u+W7IZ/Qj
MaRKkH8rAkEAw+9kS+T8zPvf7WjJpw4DCk9oWYKJTqMkeWOQ5giDxCkqMftHN/Kx
SK+tA59y4irqMXWJFVw9KF+LGc+/gCyr8QJBAMHoa2zZ8Kh9dRlM0Sn8NXmsjsja
l0dwSF51tjz/H/OUjuI1CPI8m/1DW6pJznGzlyIBNVRjrm37Tvn5uH8aSoMCQQCn
VfV/h9KLsnVdSggmnyXjkUvaXkysF1LYRTuD6jO2vT1nMGZzltbn7/lARdYU6HfY
w7vOvVcR+v9OjQnNCtnRAkEAmsU2fniVPkd9Q2OYB9ewLrR8U+5iuO2nIF1qyDRP
IQL47yI+ABP5RTZDVjNcvbrP3O1gLu+V6tC0KPsg5QIuWA==
-----END RSA PRIVATE KEY-----



```

AllegZoneBot:
```

```

Installer:
```
in /etc/nginx/conf.d/installer.conf"

server {
    listen   80;
    server_name installer.allegiancezone.com installer.spacetechnology.net installer;
    location / {
      rewrite ^ http://cdn.allegiancezone.com/install/AllegSetup_211.exe permanent;
    }
    location /latest {
      rewrite ^ http://cdn.allegiancezone.com/install/AllegSetup_211.exe permanent;
    }
    location /latest.exe {
      rewrite ^ http://cdn.allegiancezone.com/install/AllegSetup_211.exe permanent;
    }
}
```
