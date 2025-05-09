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

