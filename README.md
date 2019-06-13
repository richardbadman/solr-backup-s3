# Solr backups on AWS S3

### Back story

Where I work, we use solr v7.4 _(the hadoop version is v2.7.4)_ and we store our solr indexes on disk on bare metal. We currently create backups into a HDFS cluster, which is also on bare metal. We want to be able to store our backups on S3, with the ability to also restore from S3.

I originally followed the guide found [here](https://issues.apache.org/jira/browse/SOLR-9952), however, this presented a lot of errors for me. Originally it wants you to use the `S3N` connection, however, this is soon to be depreciated and only works for certain regions, which our buckets are not in.

I then went to convert this to `S3A` but was met with bountiful problems, people stating about the `http-client` library version, but that was fine, I forced the `shared` connection pool to `true` to see if something was closing the connection early, but again, no luck. I upgraded the hadoop to v2.8.1 using the same custom S3A class, and still no luck.

I was beginning to loose faith, until I asked if anyone had any success with this on the original ticket. This is when Kevin Risden replied of [this repo](https://github.com/risdenk/solr-s3a-testing) in which he was successful in backing up and restoring to S3 with solr v8. I tested this out, and was able to store the indexes locally and backup and restore using S3.

I was close, but just wanted 1 more thing, for this to work with the current version of solr we were using. At first this didn't work, and I thought the only thing left to try is upgrading the hadoop version _(as from a month of searching for answers, I saw a lot of people having success using hadoop v2.8+, so thought I should try this again)_, and it worked. This uses the default `HDFSBackupRepository` instead of the custom `S3A` class that is included in the original ticket as a patch.

### Using this repo

This is just to provide the patch files and an example `solr.xml` file and `core-site.xml` file.

I have included two patches:
* `hadoop-upgrade-and-aws.patch`
* `aws-only.patch`

##### `hadoop-upgrade-and-aws.patch`

This patch focuses on upgrading from hadoop v2.7.4 to hadoop v2.8.1. It could possibly work with other versions of hadoop, however, I am unsure of this. It also includes the aws packages that are also needed. Most of this was taken from [this ticket](https://issues.apache.org/jira/browse/SOLR-10951)

##### `aws-only.patch`

This patch only includes the aws dependencies that are needed for this to work.
