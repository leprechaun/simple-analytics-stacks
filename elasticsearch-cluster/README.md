# Elasticsearch Cluster

Note: this stack creates a cluster in `high-availability` mode. Which means 3 masters and a minimum of 2 data nodes. Beware of the costs.

## Resources

* ElasticsearchDomain
  This includes a complete elasticsearch cluster.

## Launching

```shell
$ stack-create ${MY_ANALYTICS_STACK_NAME} elasticsearch-cluster/elasticsearch-cluster.json [elasticsearch-cluster/elasticsearch-cluster.params.json]
```


## Parameters

* `DomainName`
  This is cluster name

* `EBSSizes`
  Defaults to 100gb. Remember that because of replication, you must multiply by two to get your desired capacity.

* `MasterNodeCount`
  For an H/A setup, you should have 3 masters.

* `MasterNodeType`
  You may need to up-size your masters, should you reach thousands of shards.

* `DataNodeCount`
  Default to 2, but you may want to increase this as your dataset grows.
  
* `DataNodeType`
  Defaults to r3.large, as memory is most valuable.

## After Launching

* Edit the Access policy of the cluster, by default, it is open to the public. The event-processor does NOT support IAM authentication, so will require you leave it open on a IP white-listing basis. This will likely be your `NAT instance`.
