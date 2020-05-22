https://github.com/kubernetes/ingress-nginx

* HTTP/s Client, like a web-browser
* Internet
* Load Balancer, often from a cloud provider, listening on port 80/443
* Kubernetes Node at a nodeport, like 20345.
* Service of type LoadBalancer. 
** This is a superset of the functionality of a service of type NodePort.
** Label Selector for the ingress-nginx pod(s)
* Ingress-Nginx pod(s), port 80/443
* nginx daemon listening on 0.0.0.0:80/443

The ingress controller consumes Ingress, ConfigMap, Secret and Service resources and creates `nginx.conf` along with some lua magic. So the connection flow doesn't really go through these resources, but they appear logically consecutive;

* Ingress resource
** HTTP-centric hostname/path rules
** Whether to teminate TLS for a given hostname, and the Secret resource with a certificate
** A backend app Service & Port
* Backend app Service's labelSelector
* All pods matching that labelSelector
** Whether those pods are Ready

The ingress controller uses that config & makes a connection directly to one of the backend app pods.

* New tcp connection from nginx daemon to the app pod
* App daemon, listening on something like 0.0.0.0:5000


Everything at the Top of this connection flow
builds on the Bottom bits.
If the bottom bits aren't working,
the top bits won't work either.

If something is not working,
start at the Bottom,
manually making requests with curl (or similar).
