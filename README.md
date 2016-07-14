# GlusterFS in Docker container

A Centos based docker container running GlusterFS daemon.

# Versions

- latest -> 3.8.1
- 3.8
- 3.7, 3.7.1
- 3.6, 3.6.1

# Usage

There you'll find several methods to launch GlusterFS with docker. The "docker-compose" method is nice to make some tests. Host method is nice to be used on real servers. 

## Real server

On each server, you may launch:

```bash
docker run --net host --privileged --name glusterfs -v /hostdata:/data  metal3d/glusterfs
```

Take a look at "`-v`" options that mounts your "/hostdata" directory inside the container as "/data". 
When you will create volumes and bricks, **you'll need to use the "container" path**. 

Note the "`--net host`" usage that will open glusterfs ports to be accessible by other nodes. 

When you're ready (having at least 2 containers running on 2 hosts), you can access glusterfs from one node and add peers. Here, we're using "node1.tld" and "node2.tld", and we're using node1 to launch commands:

```bash
# on node1.tld
$ docker exec -it glusterfs bash
/# gluster peer probe node2.tld
/# gluster create volume gv0 replica 2 transport tcp node1.tld:/data/brick1 node2.tld:/data/brick1 force
/# gluster volume start gv0
/# exit
```

Right now, we've declared a "gv0" volume that is able to be mounted on other hosts.

```bash
# NOT IN DOCKER
$ mount -t glusterfs node1.tld:/gv0 /mnt
$ date > /mnt/example.output
```

You should now have replication of `example.output` file on node1.tld and node2.tld in "`/hostdata/brick1`"

## Using docker-compose

This is a simple test that may be adapted to your needs. This example is really interessing to check how works GlusterFS and what's make options, replications, stripping, and so on.

I strongly recommand to use docker-compose "version 2" syntax that easilly configure network to let the hosts to be able to ping each others.   


This is a docker-compose.yml file that create 2 servers:

```yaml
version: '2'
services:
    gs1:
        image: metal3d/glusterfs
        privileged: true
        volumes:
        - ./volumes/server1:/data

    gs2:
        image: metal3d/glusterfs
        privileged: true
        volumes:
        - ./volumes/server2:/data
```

Then:

```bash
$ docker-compose up -d
$ docker-compose exec gs1 bash
/# gluster peer probe gs1
/# gluster volume create gv0 replica 2 gs1:/data/brick1 gs2:/data/brick1 force
/# gluster volume start gv0
/# mount gs1:/gv0 /mnt
/# date > /mnt/example.file
```

On the host, you will see "example.file" to be replicated in "`./volumes/server1/brick1`" and "`./volumes/server2/brick1`" directories

You may check logs by using:

```bash
docker-compose logs
```


