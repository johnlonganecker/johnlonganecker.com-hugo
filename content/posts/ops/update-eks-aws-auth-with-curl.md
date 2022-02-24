---
title: "Update Eks Aws Auth With Curl"
date: 2022-02-23T19:57:39-08:00
draft: true
---
# Updating EKS aws-auth config map using curl (http requests)

Tools needed
---
aws-cli
curl
jq

example code

```
cluster_name=yourClusterName
api_endpoint=$(aws eks describe-cluster --cluster-name ..)
```

Get config map
```
curl -k -XGET \
  -H "Accept: application/json" \
  -H "content-type: application/json" \
  -H "User-Agent: kubectl/v1.23.3 (linux/amd64) kubernetes/816c97a" \
  -H "accept-encoding: gzip" \
  -H "authorization: Bearer $(aws eks get-token --cluster-name $cluster_name --output_json | jq '.status.token'" \
  -d 'json' \
  $api_endpoint'/api/v1/namespaces/kube-system/configmaps?kubectl-client-side-apply'
```

Create new config map
```
curl -k XPOST \
  -H "Accept: application/json" \
  -H "content-type: application/json" \
  -H "User-Agent: kubectl/v1.23.3 (linux/amd64) kubernetes/816c97a" \
  -H "accept-encoding: gzip" \
  -H "authorization: Bearer $(aws eks get-token --cluster-name $cluster_name --output_json | jq '.status.token'" \
  -d 'json' \
  $api_endpoint'/api/v1/namespace/kube-system/configmaps/aws-auth'
```

patch existing config map
```
curl -k XPATCH \
  -H "Accept: application/json" \
  -H "content-type: application/strategic-merge-patch+json" \
  -H "User-Agent: kubectl/v1.23.3 (linux/amd64) kubernetes/816c97a" \
  -H "accept-encoding: gzip" \
  -H "authorization: Bearer $(aws eks get-token --cluster-name $cluster_name --output_json | jq '.status.token'" \
  -H "kubectl-command: kubectl patch" \
  -d 'json' \
  $api_endpoint'/api/v1/namespace/kube-system/configmaps/aws-auth'
```

Explain

get api hostname

get eks token

How did I find these values?
---
use mitmproxy

mitmproxy --ssl-insecure --listen-port 7890

You can shorten it to:

mitmproxy -k -p 7890

HTTPS_PROXY=127.0.0.0:7890 kubectl get pods --insecure-skip-tls-verify

This is for testing/learning purposes, the traffic is still encrypted, but by not verifing you run the risk of someone jumping in the middle and using their own custom tls certificate.
