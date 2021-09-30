# **How to handle "[ERR] Nodes don't agree about configuration!" when check redis cluster**

- cluster nodes info:

  ```bash
  127.0.0.1:7001> cluster nodes
  686deb15bc0ef0e86545912daa6e37fbadb3b4f6 127.0.0.1:7004@17004 master - 0 1632970649548 12 connected 5461-10922
  cadf23e1d54a15d8d3722dcc43ea4a313ab983c3 127.0.0.1:7000@17000 master - 0 1632970649347 15 connected 0-5460
  f4c14ffe14d2da4a79ac91a9ccee0267c16ad672 127.0.0.1:7003@17003 slave cadf23e1d54a15d8d3722dcc43ea4a313ab983c3 0 1632970650560 15 connected
  ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6 127.0.0.1:7002@17002 slave e46d9fcc99de48873f9a5cae53d49c97aa50ba35 0 1632970649000 14 connected
  cf3a04331c98b0ff307098345b255bdd6c5f1970 127.0.0.1:7001@17001 myself,slave 686deb15bc0ef0e86545912daa6e37fbadb3b4f6 0 1632970649000 8 connected
  e46d9fcc99de48873f9a5cae53d49c97aa50ba35 127.0.0.1:7005@17005 master - 0 1632970650359 14 connected 10923-16383
  ```
  
- briefly:
 
  ```
  4 (686deb15bc0ef0e86545912daa6e37fbadb3b4f6) -> 1 (cf3a04331c98b0ff307098345b255bdd6c5f1970): 5461-10922

  0 (cadf23e1d54a15d8d3722dcc43ea4a313ab983c3) -> 3 (f4c14ffe14d2da4a79ac91a9ccee0267c16ad672): 0-5460

  5 (e46d9fcc99de48873f9a5cae53d49c97aa50ba35) ->2 (ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6): 10923-16383
  ```
  
- err info:
 
  ```bash
  /usr/local/var/db/redis> redis-cli --cluster check 127.0.0.1:7000
  127.0.0.1:7000 (cadf23e1...) -> 0 keys | 5461 slots | 1 slaves.
  127.0.0.1:7005 (e46d9fcc...) -> 0 keys | 5461 slots | 1 slaves.
  127.0.0.1:7004 (686deb15...) -> 0 keys | 5462 slots | 1 slaves.
  [OK] 0 keys in 3 masters.
  0.00 keys per slot on average.
  >>> Performing Cluster Check (using node 127.0.0.1:7000)
  M: cadf23e1d54a15d8d3722dcc43ea4a313ab983c3 127.0.0.1:7000
     slots:[0-5460] (5461 slots) master
     1 additional replica(s)
  S: ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6 127.0.0.1:7002
     slots: (0 slots) slave
     replicates e46d9fcc99de48873f9a5cae53d49c97aa50ba35
  S: f4c14ffe14d2da4a79ac91a9ccee0267c16ad672 127.0.0.1:7003
     slots: (0 slots) slave
     replicates cadf23e1d54a15d8d3722dcc43ea4a313ab983c3
  M: e46d9fcc99de48873f9a5cae53d49c97aa50ba35 127.0.0.1:7005
     slots:[10923-16383] (5461 slots) master
     1 additional replica(s)
  S: cf3a04331c98b0ff307098345b255bdd6c5f1970 127.0.0.1:7001
     slots: (0 slots) slave
     replicates 686deb15bc0ef0e86545912daa6e37fbadb3b4f6
  M: 686deb15bc0ef0e86545912daa6e37fbadb3b4f6 127.0.0.1:7004
     slots:[5461-10922] (5462 slots) master
     1 additional replica(s)
  [ERR] Nodes don't agree about configuration!
  >>> Check for open slots...
  [WARNING] Node 127.0.0.1:7004 has slots in importing state 575,1809,3498,3687,4638,13312,14006,14016,14058,14687.
  [WARNING] The following slots are open: 14016,3687,1809,3498,13312,4638,575,14058,14006,14687.
  >>> Check slots coverage...
  [OK] All 16384 slots covered.
  ```
  
- one node's conf file seems different from others:

  ```bash
  /usr/local/var/db/redis> cat nodes-7004.conf
  e46d9fcc99de48873f9a5cae53d49c97aa50ba35 127.0.0.1:7005@17005 master - 0 1632970410183 14 connected 10923-13311 13313-14005 14007-14015 14017-14057 14059-14686 14688-16383
  ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6 127.0.0.1:7002@17002 slave e46d9fcc99de48873f9a5cae53d49c97aa50ba35 0 1632970409369 14 connected
  cf3a04331c98b0ff307098345b255bdd6c5f1970 127.0.0.1:7001@17001 slave 686deb15bc0ef0e86545912daa6e37fbadb3b4f6 0 1632970410000 12 connected
  f4c14ffe14d2da4a79ac91a9ccee0267c16ad672 127.0.0.1:7003@17003 slave cadf23e1d54a15d8d3722dcc43ea4a313ab983c3 0 1632970410183 15 connected
  cadf23e1d54a15d8d3722dcc43ea4a313ab983c3 127.0.0.1:7000@17000 master - 0 1632970410000 15 connected 0-574 576-1808 1810-3497 3499-3686 3688-4637 4639-5460
  686deb15bc0ef0e86545912daa6e37fbadb3b4f6 127.0.0.1:7004@17004 myself,master - 0 1632970410000 12 connected 5461-10922 [575-<-f4c14ffe14d2da4a79ac91a9ccee0267c16ad672] [1809-<-f4c14ffe14d2da4a79ac91a9ccee0267c16ad672] [3498-<-f4c14ffe14d2da4a79ac91a9ccee0267c16ad672] [3687-<-f4c14ffe14d2da4a79ac91a9ccee0267c16ad672] [4638-<-f4c14ffe14d2da4a79ac91a9ccee0267c16ad672] [13312-<-ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6] [14006-<-ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6] [14016-<-ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6] [14058-<-ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6] [14687-<-ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6]
  vars currentEpoch 15 lastVoteEpoch 15
  ```
  
- kill this node
  
- change it's conf to:
 
  ```bash
  /usr/local/var/db/redis  cat nodes-7004.conf
  cf3a04331c98b0ff307098345b255bdd6c5f1970 127.0.0.1:7001@17001 master - 0 1632972060964 16 connected 5461-10922
  cadf23e1d54a15d8d3722dcc43ea4a313ab983c3 127.0.0.1:7000@17000 master - 0 1632972060964 15 connected 0-5460
  e46d9fcc99de48873f9a5cae53d49c97aa50ba35 127.0.0.1:7005@17005 master - 1632972060963 1632972060960 14 connected 10923-16383
  f4c14ffe14d2da4a79ac91a9ccee0267c16ad672 127.0.0.1:7003@17003 slave cadf23e1d54a15d8d3722dcc43ea4a313ab983c3 0 1632972060964 15 connected
  ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6 127.0.0.1:7002@17002 slave e46d9fcc99de48873f9a5cae53d49c97aa50ba35 1632972060963 1632972060960 14 connected
  686deb15bc0ef0e86545912daa6e37fbadb3b4f6 127.0.0.1:7004@17004 myself,slave cf3a04331c98b0ff307098345b255bdd6c5f1970 0 1632972060960 12 connected
  vars currentEpoch 16 lastVoteEpoch 15
  ```
  
 - restart this node
 
 - check again:
 
  ```bash
  /usr/local/var/db/redis  redis-cli --cluster check 127.0.0.1:7001
  127.0.0.1:7001 (cf3a0433...) -> 0 keys | 5462 slots | 1 slaves.
  127.0.0.1:7000 (cadf23e1...) -> 0 keys | 5461 slots | 1 slaves.
  127.0.0.1:7005 (e46d9fcc...) -> 0 keys | 5461 slots | 1 slaves.
  [OK] 0 keys in 3 masters.
  0.00 keys per slot on average.
  >>> Performing Cluster Check (using node 127.0.0.1:7001)
  M: cf3a04331c98b0ff307098345b255bdd6c5f1970 127.0.0.1:7001
  slots:[5461-10922] (5462 slots) master
  1 additional replica(s)
  S: 686deb15bc0ef0e86545912daa6e37fbadb3b4f6 127.0.0.1:7004
  slots: (0 slots) slave
  replicates cf3a04331c98b0ff307098345b255bdd6c5f1970
  M: cadf23e1d54a15d8d3722dcc43ea4a313ab983c3 127.0.0.1:7000
  slots:[0-5460] (5461 slots) master
  1 additional replica(s)
  S: f4c14ffe14d2da4a79ac91a9ccee0267c16ad672 127.0.0.1:7003
  slots: (0 slots) slave
  replicates cadf23e1d54a15d8d3722dcc43ea4a313ab983c3
  S: ba4cf1ff5dfa5c002bdcedd026279bc3b6d3d2e6 127.0.0.1:7002
  slots: (0 slots) slave
  replicates e46d9fcc99de48873f9a5cae53d49c97aa50ba35
  M: e46d9fcc99de48873f9a5cae53d49c97aa50ba35 127.0.0.1:7005
  slots:[10923-16383] (5461 slots) master
  1 additional replica(s)
  [OK] All nodes agree about slots configuration.
  >>> Check for open slots...
  >>> Check slots coverage...
  [OK] All 16384 slots covered.
  ```
