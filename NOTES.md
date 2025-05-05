Load Balancing Examples:

+ HAProxy(tcp and http)
+ nginx(tcp and http)
+ Apache(mod_proxy_balancer, http only)
+ F5
+ Cloud-based load balancer


Load Balancing Algorithms:

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

stats Webpage

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




