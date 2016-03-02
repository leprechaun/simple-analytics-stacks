# Event Processor

## Resources

### AutoscalingGroup + LaunchConfiguration

We need to have an instance running, and get 3 docker containers running.

### IAM Policy

Our instance needs a set of permissions to do it's thing.

* Read/Write on the S3 buckets.
* Read/Write on the Kinesis stream
* Read/Write on dynamodb tables (for Kinesis synchronisation)

## Parameters

The first half of the parameters are the outputs of the `cloudfront-buckets-and-stream`.

* `S3BucketRawAccessLogs`
  The bucket where Cloudfront access logs are stored.

* `S3BucketCleanedEvents`
  The bucket where we will copy cleaned/processed events for archival.

* `KinesisEventStream`
  The name of the Kinesis stream

* `ElasticsearchEndpoint`
  In the form of `my-elasticsearch-cluster.somewhere.com:9200`

AWS EC2 specific parameters

* `Subnets`
  Comma seperated list of subnets in which to place the instance.
  They required internet connectivity.

* `KeyName`
  An SSH key-pair name to use.

* `ImageId`
  An EC2 AMI id. This stack was only tested on Amazon linux (and derivatives)

* `InstanceType`
  Type, or size, of the instance. You may need to scale up, depending on your event volume.

* `VpcId`
  The VPC in which your instance/subnet resides.

## After Launching

* The files in S3BucketRawAccessLogs should get moved into the `Processed/` prefix.

* The Kinesis stream should register reads and writes

* Your Elasticsearch cluster should begin receiving data.

# Under the hood

This stack's responsibility is to launch 3 docker containers.

* Access log reader (https://hub.docker.com/r/leprechaun/docker-logstash-input-s3-frontend-output-kinesis/)
* S3 archiver (https://hub.docker.com/r/leprechaun/docker-logstash-input-kinesis-output-s3-archiver/)
* Elasticsearch indexer (https://hub.docker.com/r/leprechaun/docker-logstash-input-kinesis-output-es-indexer/)

