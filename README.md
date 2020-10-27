# Run Locally

```shell
$ docker-compose up -d

$ docker container ls

CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                            NAMES
3e4d5deece3b        es-sample_elasticsearch   "/usr/local/bin/dock…"   7 minutes ago       Up 7 minutes        0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   es-sample_elasticsearch_1

$ curl localhost:9200
{
  "name" : "3e4d5deece3b",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "Tn5dIN5zT6yztExTNaZblQ",
  "version" : {
    "number" : "7.4.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "2f90bbf7b93631e52bafb59b3b049cb44ec25e96",
    "build_date" : "2019-10-28T20:40:44.881551Z",
    "build_snapshot" : false,
    "lucene_version" : "8.2.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```