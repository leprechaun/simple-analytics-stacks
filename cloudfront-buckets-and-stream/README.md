# Cloudfront, Buckets & Stream

This stack provides the pre-requisites for the rest of the system.
They are also the ones least likely to change.

## Resources

* S3BucketRawAccessLogs
    An S3 bucket where access logs will be stored.

* S3BucketCleanedEvents
    An S3 bucket where a copy of cleaned/processed events will be archived.

* CloudfrontDistribution
    A Cloudfront distribution configured for caching and access logging.
    

## Launching

```shell
$ stack-create $NAME \
 cloudfront-buckets-and-stream/cloudfront-buckets-and-stream.json
```

Do note that the stack name you specify will be used as part of the S3 bucket names.

## After launching

You will need to place the gif file at the root of the bucket. The access
policy will allow for any name which starts with `track`. You may change the access policy if this does not suit you.

## Testing

Make a few requests against the tracking pixel (https://XYZ123456.cloudfront.net/track.gif) and make sure the access logs appear in the logs bucket.
