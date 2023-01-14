---
title: Kubernetes Multiple Watches Using Threads in Python
categories:
- Python
- Kubernetes
feature_image: "https://miro.medium.com/max/1400/1*5WEua14oixcXrIi7Tuqgfg.webp"
---
---

Kubernetes Multiple Watches Using Threads in Python
In this post, I am going to talk about How To Make Multiple Watches for Kubernetes Cluster in Python.
I was facing to make Multiple watch events for Kubernetes. By using Kubernetes Python-Client, it is easy to make Watch Event using kubernetes.```watch.Watch()```.
Single Watch for Kubernetes Cluster In Python

```
from kubernetes import client, config, watch

config.load_kube_config()

api_instance = client.CoreV1Api()

w = watch.Watch()
for event in w.stream(api_instance.list_pod_for_all_namespaces, timeout_seconds=0):
    print("Event: %s %s %s" % (event['type'], event['object'].kind, event['object'].metadata.name))
```

You can see the result here.
It shows all the pods in the cluster with ADDED type.
However, when I add another for loop to watch namespaces, it does not work.
```
from kubernetes import client, config, watch

config.load_kube_config()

api_instance = client.CoreV1Api()

w = watch.Watch()
##First Loop for watching
for event in w.stream(api_instance.list_pod_for_all_namespaces, timeout_seconds=0):
    print("Event: %s %s %s" % (event['type'], event['object'].kind, event['object'].metadata.name))
##Second Loop for watching
for event in w.stream(api_instance.list_namespace, timeout_seconds=0):
    print("Event: %s %s %s" % (event['type'], event['object'].kind, event['object'].metadata.name))
```
It is not working because it uses a single thread, and the thread is running on the first loop, so the second loop for watching is not running.

---

The purpose of this post it to make it works.
How do we make it works?
We can use multiple threads and make it works.
The idea is that one watch event use a thread, so if you have multiple watch events, then you can use more threads to handle it.
Here is the sample code.
```
from kubernetes import client, config, watch
from kubernetes.config import ConfigException
from urllib3.exceptions import ProtocolError
import concurrent.futures
try:
# Load configuration inside the Pod
   config.load_incluster_config()
except ConfigException:
# Load configuration for testing
   config.load_kube_config()
api_instance = client.CoreV1Api()
def namespaces():
   w = watch.Watch()
   try:
      for event in   w.stream(api_instance.list_namespace,timeout_seconds=0):
         
         print("Event: %s %s %s" % (event['type'],event['object'].kind, event['object'].metadata.name))
   except ProtocolError:
      print("watchPodEvents ProtocolError, continuing..")
def pods():
   w = watch.Watch()
   try:
      for event in w.stream(api_instance.list_pod_for_all_namespaces,timeout_seconds=0):
         print("Event: %s %s %s" % (event['type'],event['object'].kind, event['object'].metadata.name))
   except ProtocolError:
      print("watchPodEvents ProtocolError, continuing..")
if __name__ == "__main__":
with concurrent.futures.ThreadPoolExecutor() as executor:
   n = executor.submit(namespaces)
   p = executor.submit(pods)
```
I used concurrent.futures and concurrent.futures.ThreadPoolExecuter to make it running with multiple threads.
So the result is that you made multiple watches for Kubernetes Clusters!