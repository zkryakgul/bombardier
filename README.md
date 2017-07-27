# bombardier

Bombardier docker container with some ip tools for using with openvswitch

Pre info:
br0 = openvswitch bridge

Run container and add ovs port to it :
```
for i in `seq 0 10`;do docker run -d --privileged -c 0 --net=none --cap-add NET_ADMIN --name bomb$i -it zkryakgul/bombardier &&\
ovs-docker add-port br0 eth0 bomb$i --ipaddress=10.10.10.$(($i+50))/24 && \
docker exec -d -it bomb$i /sbin/ethtool -K eth0 tx off rx off 
done
```
start bombardier:
```sh
for i in `seq 0 10`;do
docker exec -d -it bomb$i sh -c "bombardier -c 16 -d 100s http://10.10.10.6:81/ > /usr/output"
done
```

show log files:
```sh
for i in `seq 0 10`;do 
docker exec -it bomb$i /bin/cat /usr/output
done
```
show total req per second:
```sh
for i in `seq 0 10`;do  docker exec -it bomb$i /bin/cat /usr/output; done | grep Reqs | awk 'BEGIN {FS=" "}{print $2}' | paste -sd+ | bc
```
delete ports and containers:
```sh
for i in `seq 0 10`;do 
ovs-docker del-port br0 eth0 bomb$i && docker rm -f bomb$i 
done
```
