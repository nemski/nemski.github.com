---
layout: post
title: Speeding up AWS Lambda functions with Python Cachetools
date: 2017-07-02 00:00:00
---

This is a quick post to talk about a technique I've been using while writing Python functions for AWS Lambda. Often the slowest part of my functions are API calls to AWS services like STS and DynamoDB. We use STS to assume roles in other AWS Accounts, but unlike other SDKs Boto doesn't provide an Assume Role Provider, which would cache the credentials for you. We often use Dynamo as a backing store for data used in our Lambda functions, data that rarely changes.

This mechanism makes use of a Python library called [Cachetools](https://pypi.python.org/pypi/cachetools), as well as the inbuilt Lambda runtime cache. In order for this technique to work your Lambda function must stay "warm", there are many posts available to help you understand how this works so I won't go into it here.

Lambda functions run in AWS by [reusing containers](https://aws.amazon.com/blogs/compute/container-reuse-in-lambda/) across individual runs. This speeds up the Lambda start time and has the added benefit of caching the python runtime process, which allows this technique to work.

The following code works for Python 3.6 runtime, but can easily be adapted for Python 2.7.

First we import the Cachetools library:

```
import cachetools.func
```

In this example we have a class method that retrieves some data from a Dynamo table called "accounts". By providing the cachetools function decorator (in this example a TTL cache, but we could also use the LRU or LFU algorithm) we memoize the function call, causing it to return the cached value if the parameter (the account number) is the same:

```
    @classmethod
    @cachetools.func.ttl_cache(maxsize=64, ttl=300)
    def get_account(cls, account_number):
      dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
      table = dynamodb.Table("accounts")
```

The cache lasts for 5 minutes, ensuring we eventually retrieve an up to date value if it does change. This not only speeds up our function by several hundred milliseconds, it allows us to keep our DynamoDB Provisioned Reads lower and hence our costs lower.

Note though that the cache takes up Memory in your Lambda function, so you may see it grow over time and eventually eat up all your available RAM.
