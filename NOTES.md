## Load Balancing Examples:

+ HAProxy(tcp and http)
+ nginx(tcp and http)
+ Apache(mod_proxy_balancer, http only)
+ F5
+ Cloud-based load balancer


## Load Balancing Algorithms:

+ round  robin
+ static-rr(round robin)
+ leastconn
+ first
+ source

What is a Frontend?

+ Configuration block
+ Exposes IPs/ports
+ Contains settings
+ Performs actions
+ Directs traffic to backend


What is a Backend?

+ Configuration block
+ Load balancing
+ server
+ stick-table

stats interface

+ UNIX socket
+ Can use netcat(nc) forcommunication
  + Non-interactive mode
  + interactive mode

## stats Webpage

+ Displays HAProxy statics
+ Organized per-frontend/backend/server
+ Restrict access via HAProxy configuration
+ downsides
  + Static
  + Ephemeral

+ ACLs - Perform 1 or more actions based on a condition
+ MAPs - Lockup table that stores key-value pairs
+ Stick tables - Key-value store for tracking counts of certain items


What Are Stick Tables?

+ Key-value store
+ Key is what you're tracking(example: client IP)
+ Value counts what you're tracking(example: client IP requests in the pass 10 seconds) 
  + Can track a large variety of items


## lab env

```bash

yum -y module install container-tools
yum install -y epel
yum install -y figlet
mkdir ~/testfiles
for site in `seq 1 2`; do for server in `seq 1 3`; do figlet -f big SITE$site - WEB$server > ~/testfiles/site$site\_server$server.txt;done;done

port=1;for site in `seq 1 2`; do for server in `seq 1 3`; do docker run -dt --name site$site\_server$server -p 800$(($port)):80 nginx;port=$(($port+1));done;done
for site in `seq 1 2`; do for server in `seq 1 3`; do docker cp ~/testfiles/site$site\_server$server.txt site$site\_server$server:/usr/share/nginx/html/test.txt;done;done
port=1;for site in `seq 1 2`; do for server in `seq 1 3`; do curl http://127.0.0.1:800$port/test.txt nginx;port=$(($port+1));done;done
```

> yum install haproxy

> setsebool -P haproxy_connect_any 1

> vim /etc/haproxy/haproxy.cfg


```
# Site-Frontends
frontend site1
    bind *:8000
    default_backend site1

frontend site2
    bind *:8100
    default_backend site2

# Site 1 backend
backend site1
    balance roundrobin
    server site1-web1 127.0.0.1:8001 check
    server site1-web2 127.0.0.1:8002 check
    server site1-web3 127.0.0.1:8003 check

# Site 2 backend
backend site2
    balance roundrobin
    server site2-web1 127.0.0.1:8004 check
    server site2-web2 127.0.0.1:8005 check
    server site2-web3 127.0.0.1:8006 check

# Stats Page
listen stats
    bind *:8050
    stats enable
    stats uri /
    stats hide-version

```

```
systemctl enable --now haproxy
curl localhost:8000/test.txt
curl localhost:8100/test.txt

# stop all containers
docker stop $(docker ps -q)

# start all containers
docker start $(docker ps -q -a)

podman start site{1..2}_server{1..3}

docker start site{1..2}_server{1..3}


```

## http rewrite

HTTP Rewrites

+ Change the HTTP request method
+ Manipulate HTTP headers
+ Set the URL path
+ Set the query string
+ Set the URI

> curl -s http://127.0.0.1:8000/test.txt
> curl -s http://127.0.0.1:8100/test.txt

```bash

for site in `seq 1 2`; do for server in `seq 1 3`; do docker exec site$site\_server$server mkdir /usr/share/nginx/html/textfiles;done;done

for site in `seq 1 2`; do for server in `seq 1 3`; do docker exec site$site\_server$server mv -v /usr/share/nginx/html/test.txt /usr/share/nginx/html/textfiles;done;done


docker exec -it site1_server1 /bin/bash

http http://127.0.0.1:8000/textfiles/test.txt

```

```
frontend site1
    bind *:8000
    default_backend site1
    acl p_ext_txt path_end -i .txt
    acl p_folder_textfiles path_beg -i /textfiles/
    http-request set-path /textfiles/%[path] if !p_folder_textfiles p_ext_txt
```

## Load Balancing HTTPS 

### SSL/TLS Support

Types:

+ SSL paththrough
+ SSL termination
  + Single point of management
  + Offload SSL processing


```
front-end http-in
    bind *:80
    mode http
    acl site-1 hdr(host) -i www.site1.com
    acl site-2 hdr(host) -i www.site2.com
    use_backend site1 if site-1
    use_backend site2 if site-2


# change host dns

> 127.0.0.1 www.site1.com
> 127.0.0.1 www.site2.com

curl http://www.site1.com/textfiles/test.txt
curl http://www.site2.com/textfiles/test.txt
```

```
cd /etc/haproxy/certs/

openssl genrsa -out site1.key 2048
openssl req -new -x509 -key site1.key -out site1.crt -days 365 -subj "/CN=site1.com"

cat site1.key site1.crt > site1.pem

openssl genrsa -out site2.key 2048
openssl req -new -x509 -key site2.key -out site2.crt -days 365 -subj "/CN=site2.com"

cat site2.key site2.crt > site2.pem

front-end http-https-in
    bind *:80
    bind *:443 ssl crt /etc/haproxy/certs/site1.pem  crt /etc/haproxy/certs/site2.pem force-tlsv12
    mode http
    http-request redirect scheme https unless { ssl_fc }
    acl site-1 hdr(host) -i www.site1.com
    acl site-2 hdr(host) -i www.site2.com
    use_backend site1 if site-1
    use_backend site2 if site-2

curl -k https://www.site1.com/textfiles/test.txt
curl -L -k http://www.site1.com/textfiles/test.txt
```

## Configuring DDoS Attack protection 

Dealing with Attachs

Common Types:

+ HTTP Flood
  + Overwhelm site with with request volumn
+ Slowloris
  + Slow requests - tie up connections
+ Static characteristics
  + ACLs
  + Blocking by IP address

| SOFTWARE |
| ------------- |
| lAYER7 APPLICATION  UPPER LAYER DATA HTTP,SSH, NFS, RDP, NTP, SMTP |
| lAYER6 PRESENTATION UPPERLAYER DATA SSL, TLS, ASCII, EBCDIC, ENCRYPT|
| LAYER 5 SESSION  UPPERLAYER DATA NETBIOS, NCP, PPTP, RPC, SCP |
| LAYER 4 TRANSPORT SEGMENT TCP, UDP, ATP, SCTP, SPX |
| LAYER 3 NETWORK PACKET IPOV4, IPV6, ICMP, IGMP, IPSEC, IPX |
| HARDWARE |
| LAYER 2 DATALINK FRAME ARP, ETHERNET, MPLS, MAC, PPP |
| LAYER 1 PHYSICAL BITS, CABLING,DSL, T1, GSM, MODEM |


### What is a Named ACL

+ Assign a condition to name in ACL, (acl is_iamge path -i -m beg /images/)
+ Use with if and unless statements to perform 1 or more actions

### What is an In-Line ACL

+ Combines the condition check with if and unless statements to perform a single action
+ use_backend be_site3 if {path -i -m beg /images/}
+ More efficient for single-use cases

### What Are Haproxy Maps

+ Plain text
+ Stores key-value pairs
+ Used as a lookup table
+ Can be edited directly, via API, or using http-request set-map

### What are Stick Tables

+ Key-value store
+ Key is what you are tracking(example: client IP)
+ Value counts what you are tracking(example, client IP requests in  the past 10 seconds)
+ Can track a large variety of items
+ Relies heavily on HAProxy ACLs
+ Only 1 stick table per frontend of backend


> ab -n 100000 -c 100 http://www.site1.com > ~/ab.site1.log > /dev/null 2>&1 &
> ab -n 100000 -c 100 http://www.site2.com > ~/ab.site2.log > /dev/null 2>&1 &


```
touch /etc/haproxy/blocked.acl

frontend http-https-in
    bind *:80
    bind *:443 ssl crt /etc/haproxy/certs/site1.pem  crt /etc/haproxy/certs/site2.pem force-tlsv12
    mode http
    http-request redirect scheme https unless { ssl_fc }
    http-request track-sc0 src table per_ip_rates
    http-request deny deny_status 429 if { sc_http_req_rate(0) gt 100 }
    http-request deny deny_status 500 if { req.hdr(user-agent) -i -m sub curl }
    http-request deny deny_status 503 if { src -f /etc/haproxy/blocked.acl }

    acl site-1 hdr(host) -i www.site1.com
    acl site-2 hdr(host) -i www.site2.com
    use_backend site1 if site-1
    use_backend site2 if site-2

# backend for per_ip_rates
backend per_ip_rates
    stick-table type ip size 1m expire 10m store http_req_rate(10s)

wget --no-check-certificate -O -  http://www.site1.com/textfiles/test.txt
wget --no-check-certificate -O -  http://www.site2.com/textfiles/test.txt

```




