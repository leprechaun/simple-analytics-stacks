# Simple Analytics

## Our problem

Our organisation has been a longtime user of a proprietary user analytics service,
which to this day serves as the authoritative source of web analytics.

However, the product and its data-flow are blackboxes we cannot inspect or tap into.
The interface is oftentimes more complicated that it should, and granting access
to users in our organisation has been troublesome due to our size.

This led to the majority of people in our organisation with little to no visibility
as to how our users interact with our applications. We needed to change that.

## Our solution

We initially investigated the ELK stack as a log processing platform, but our
biggest win so far as been in augmenting our frontend/browser analytics system.

It's a 3 part solution.

### Data Capture

Data capture is done through simple HTTP access logs. Any JSON string included
as part of the HTTP query string will be passed to the rest of the system.

An AWS S3 bucket hosts a clear 1x1 pixel, which is served by an AWS Cloudfront
distribution (cost-effective CDN). Both of these services require no maintenance
and "will never go down."

AWS Cloudfront then write the access logs to a second AWS S3 bucket, and made
available for reading by other systems.

### Data ingestion (part 1)

Logstash is used to read the raw access logs stored in the AWS S3 bucket, extract
the JSON payload, and write that to a AWS Kinesis stream.

### Data stream

AWS Kinesis is a hosted data stream service. You pay for provisionned read and
write throughput. At our scale, it has proved more cost effective than self-hosting
alternatives (like Apache Kafka).

AWS Kinesis is the 'fan-out' mecanism. Any number of clients can write to a stream,
whether they are the same or different applications. Any number of clients can
read data from a stream, whether they are the same or different applications
(data partitioning is automatically handled by the provided AWS Kinesis libraries).

Another key feature of AWS Kinesis is that a client reading from a stream can
at any time stop and rewind and re-process data. By default, the window is 24,
but can be increased to 7 days at minimal extra cost.

### Data ingestion (part 2)

Logstash is then used again, twice, to read from the same AWS Kinesis stream.

First, the stream is copied to an AWS S3 bucket where events are cleanly JSON
formatted.

Second, the stream is also indexed into an Elasticsearch cluster, where the
data will be available for visualisations and analysis through Kibana.

## Deployment

This repository contains 2 AWS Cloudformation templates.

### Data Capture

First is the data capture stack, which provides the Cloudfront distribution,
AWS Kinesis stream and AWS S3 buckets.

It can be launched with the AWS command-line tools.

```shell
$ aws cloudformation create-stack \
 --stack-name $MY_ANALYTICS_STACK_NAME \
 --template-body file://cloudfront-buckets-and-stream.json \
 --capabilities CAPABILITY_IAM
```

An example of the parameters is provided.

### Data ingestion

Second is an AutoScalingGroup that runs 3 docker containers.

* Access log reader (https://hub.docker.com/r/leprechaun/docker-logstash-input-s3-frontend-output-kinesis/)
* S3 archiver (https://hub.docker.com/r/leprechaun/docker-logstash-input-kinesis-output-s3-archiver/)
* Elasticsearch indexer (https://hub.docker.com/r/leprechaun/docker-logstash-input-kinesis-output-es-indexer/)

It can be launched with the AWS command-line tools.

```shell
$ aws cloudformation create-stack \
 --stack-name $MY_ANALYTICS_STACK_NAME \
 --template-body file://event-processor.json \
 --parameters file://event-processor.params.json \
 --capabilities CAPABILITY_IAM
```

### Elasticsearch

For the time being, the provisioning and securing of an Elasticsearch cluster
is left as an exercise for the reader. An easy alternative is to use AWS's
Elasticsearch service. While it's not perfect, it remains much easier than
hosting your own.

## Thanks

I wish to thank my employer/client who has graciously let me make this public.
