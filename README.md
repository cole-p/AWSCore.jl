# AWSCore

Amazon Web Services Core Functions and Types.

See seperate modules for service interfaces:

 - [AWS S3](http://github.com/samoconnor/AWSS3.jl)
 - [AWS SQS](http://github.com/samoconnor/AWSSQS.jl)
 - [AWS SNS](http://github.com/samoconnor/AWSSNS.jl)
 - [AWS IAM](http://github.com/samoconnor/AWSIAM.jl)
 - [AWS EC2](http://github.com/samoconnor/AWSEC2.jl)
 - [AWS Lambda](http://github.com/samoconnor/AWSLambda.jl)


### Features

AWS Signature Version 4.

Automatic HTTP request retry with exponential back-off.

Parsing of XML and JSON API error messages to AWSException type.

Automatic API Request retry in case of ExpiredToken or HTTP Redirect.



### Configuration

Most `AWSCore` functions take a configuration object `aws` as the first argument.

A default configuration can be obtained by calling `aws_config()`. e.g.:

```julia
aws = aws_config()
or
aws = aws_config(region = "ap-southeast-2")
```

The `aws_config()` function attempts to load AWS credentials from:

 - EC2 Instance Credentials,
 - AWS Lambda Role Credentials (i.e. `env["AWS_ACCESS_KEY_ID"`), or
 - `~/.aws/credentials`

A `~/.aws/credentials` file can be created using the
[AWS CLI](https://aws.amazon.com/cli/) command `aws configrue`.
Or a `~/.aws/credentials` file can be created manually:

```ini
[default]
aws_access_key_id = AKIAXXXXXXXXXXXXXXXX
aws_secret_access_key = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

An `aws` configuration object can also be created directly from a key pair
as follows. However, putting access credentials in source code is discouraged.

```julia
aws = aws_config(creds = AWSCredentials("AKIAXXXXXXXXXXXXXXXX",
                                        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"))
```


### Exceptions

May throw: UVError, HTTPException or AWSException.


### Examples


Create an S3 bucket and store some data...

```julia
s3_create_bucket(aws, "my.bucket")
s3_enable_versioning(aws, "my.bucket")

s3_put(aws, "my.bucket", "key", "Hello!")
println(s3_get(aws, "my.bucket", "key"))
```


Post a message to a queue...

```julia
q = sqs_get_queue(aws, "my-queue")

sqs_send_message(q, "Hello!")

m = sqs_receive_message(q)
println(m["message"])
sqs_delete_message(q, m)
```


Post a message to a notification topic...

```julia
sns_create_topic(aws, "my-topic")
sns_subscribe_sqs(aws, "my-topic", q; raw = true)

sns_publish(aws, "my-topic", "Hello!")

m = sqs_receive_message(q)
println(m["message"])
sqs_delete_message(q, m)

```


Start an EC2 server and fetch info...

```julia
ec2(aws, "StartInstances", {"InstanceId.1" => my_instance_id})
r = ec2(aws, "DescribeInstances", {"Filter.1.Name" => "instance-id",
                                   "Filter.1.Value.1" => my_instance_id})
println(r)
```


Create an IAM user...

```julia
iam(aws, "CreateUser", {"UserName" => "me"})
```


Get a list of DynamoDB tables...

```julia
r = dynamodb(aws, "ListTables", "{}")
println(r)
```
