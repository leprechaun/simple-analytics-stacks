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

This repository contains 3 AWS Cloudformation templates.

The `bash-my-aws` bash helpers for AWS are included as a git submodule.
Run `git submodule init && git submodule update` to pull a copy.
Then, `source bash-my-aws/lib/stack-functions`

Or don't, and just use the python based aws command line tools directly.

### cloudfront-buckets-and-stream

[cloudfront-buckets-and-stream](cloudfront-buckets-and-stream/README.md)

### event-processor

[event-processor](event-processor/README.md)

### elasticsearch

[elasticsearch-cluster](elasticsearch-cluster/README.md)

### Data Capture

## Thanks

I wish to thank my employer/client who has graciously let me make this public.
