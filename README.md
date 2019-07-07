Nomad podman Driver
==================

*THIS IS A PROOF OF CONCEPT PLUGIN*. Do not run it in production!
Contributions are welcome, of course.

== Building The Driver ==

This project has a go.mod definition. So you can clone it to whatever directory you want.
It is not necessary to setup a go path at all.

```sh
$ git clone git@github.com:pascomnet/nomad-driver-podman
cd nomad-driver-podman
./build.sh
```

== Runtime dependencies ==

- [Nomad](https://www.nomadproject.io/downloads.html) 0.9+
- Linux host with `podman` installed

You need a varlink enabled podman binary and a system socket activation unit,
see https://podman.io/blogs/2019/01/16/podman-varlink.html. 

nomad agent, nomad-driver-podman and podman will reside on the same host, so you 
do not have to worry about the ssh aspects of podman varlink.

Ensure that nomad can find the plugin, see [plugin_dir](https://www.nomadproject.io/docs/configuration/index.html#plugin_dir)

== Using the driver ==

The featureset is very limit. 
For now you can:

* use the jobs driver config to define the image for your container
* start/stop containers. 
* use nomad alloc data in the container. It's bind mounted to /nomad
* monitor the memory consuption but not yet the cpu

Example job:

```
job "redis" {
  datacenters = ["dc1"]
  type        = "service"

  group "redis" {
    task "redis" {
      driver = "podman"

        config {
                image = "docker://redis"
        }

      resources {
        cpu    = 500
        memory = 256
      }
    }
  }
}
```

```sh
nomad run redis.nomad

==> Monitoring evaluation "9fc25b88"
    Evaluation triggered by job "redis"
    Allocation "60fdc69b" created: node "f6bccd6d", group "redis"
    Evaluation status changed: "pending" -> "complete"
==> Evaluation "9fc25b88" finished with status "complete"

podman ps

CONTAINER ID  IMAGE                           COMMAND               CREATED         STATUS             PORTS  NAMES                                                                              
6d2d700cbce6  docker.io/library/redis:latest  docker-entrypoint...  16 seconds ago  Up 16 seconds ago         redis-60fdc69b-65cb-8ece-8554-df49321b3462
```
