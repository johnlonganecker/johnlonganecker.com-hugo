---
title: "Update Eks Aws Auth With Curl"
date: 2022-02-23T19:57:39-08:00
draft: true
---
# Updating EKS aws-auth config map using curl (http requests)

### Problem
I want to give additional IAM Users/Roles access to an EKS Cluster with minimal amount of work and dependencies.

### Solution
Use aws cli to get get a token to use with an HTTP request that updates the aws-auth config map with the additional users.

### Why would you want to do this?
You might be in a situation where you want to use a minimal set of tools to add more users/roles to your EKS cluster. For example you native lambda functions with boto3 + requests to update your aws-auth config map.

Tools needed
---
aws cli
curl
jq

example code

```
cluster_name=yourClusterName
api_endpoint=$(aws eks describe-cluster --name $cluster_name | jq '.cluster.endpoint' | sed 's/"//g')
aws_auth_json='{
    "apiVersion": "v1",
    "data": {
        "mapRoles": "- rolearn: <ARN of instance role (not instance profile)>\n  username: system:node:{{EC2PrivateDNSName}}\n  groups:\n    - system:bootstrappers\n    - system:nodes\n"
    },
    "kind": "ConfigMap",
    "metadata": {
        "name": "aws-auth",
        "namespace": "kube-system"
    }
}'

Get config map
```
curl -k -XGET \
  -H "Accept: application/json" \
  -H "authorization: Bearer $(aws eks get-token --cluster-name $cluster_name --output=json | jq '.status.token' | sed 's/\"//g')" \
  -H "accept-encoding: gzip" \
  $api_endpoint'/api/v1/configmaps?limit=500'
```

Create new config map
```
curl -k -XPOST \
  -H "Accept: application/json" \
  -H "content-type: application/json" \
  -H "accept-encoding: gzip" \
  -H "authorization: Bearer $(aws eks get-token --cluster-name $cluster_name --output=json | jq '.status.token' | sed 's/\"//g')" \
  -d $aws_auth_json \
  $api_endpoint'/api/v1/namespaces/kube-system/configmaps'
```

patch existing config map
```
curl -k -XPATCH \
  -H "Accept: application/json" \
  -H "content-type: application/strategic-merge-patch+json" \
  -H "accept-encoding: gzip" \
  -H "authorization: Bearer $(aws eks get-token --cluster-name $cluster_name --output=json | jq '.status.token' | sed 's/\"//g')" \
  -d $aws_auth_json \
  $api_endpoint'/api/v1/namespaces/kube-system/configmaps/aws-auth'
```

Explain

get api hostname

get eks token

How did I find these values?
---
use mitmproxy

```
mitmproxy --ssl-insecure --listen-port 7890
```

You can shorten it to:

```
mitmproxy -k -p 7890
```

```
HTTPS_PROXY=127.0.0.0:7890 kubectl get pods --insecure-skip-tls-verify
```

This is for testing/learning purposes, the traffic is still encrypted, but by not verifing you run the risk of someone jumping in the middle and using their own custom tls certificate.
