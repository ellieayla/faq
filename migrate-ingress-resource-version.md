## Q:

When kubernetes deprecates an api version (ex: networking.k8s.io/v1beta1 -> networking.k8s.io/v1),
do I need to create new resources under the new version or migrate the old resources?

I note that two different versions (eg Ingress in v1beta1 vs v1) can have different schema.


## Short answer

No.

Your kubernetes api-server is already serving multiple versions of the same resource _today_.

You just need to stop reading/writing/documenting the deprecated versions.

That means update yaml in your git repos and wiki pages. And if you wrote code to interact with the kubernetes api-server, update your code.


## Analogy

A good way to think about a resource being available in multiple api-groups and with multiple schemas just to consider the Format of the resource.
Consider a Person resource in some pretend API, available in XML, JSON, and CSV. But this API has no Content-Type negotiation, and no Versions.
Instead, the client specifies the format it wants in the url, like `/person/<id>.<format>`.

* /person/12345.json
* /person/12345.xml
* /person/12345.csv

This person 12345 is probably stored in a database somewhere, not in JSON or XML or CSV.
When a client requests a person in some format, the webservice reads a row from a `persons` table and serializes it into the requested format.
When a client updates a person in some format, the webservice parses the submitted format and upserts a row in a `persons` table.

Someone updating person 12345's favourite color would _not_ be expected to update multiple formats.

The webservice deprecating the CSV format & url means a client needs to stop reading/writing/documenting `/person/<whatever>.csv`.

This is key: The client does _not_ need to read all the people out in CSV and create new JSON people.


## Ingress resources

The Kubernetes api-server follows a similar pattern. The api-version in the url defines a format-structure.

```
/apis/networking.k8s.io/v1/namespaces/default/ingresses/awesome
```

For example, an Ingress resource named `awesome` in namespace `default` can be accessed at multiple urls,
in multiple api versions, with multiple schemas.

```
kubectl -n default get ingresses.v1beta1.extensions -v=6
  /apis/extensions/v1beta1/namespaces/default/ingresses

kubectl -n default get ingresses.v1beta1.networking.k8s.io -v=6
  /apis/networking.k8s.io/v1beta1/namespaces/default/ingresses

kubectl -n default get ingresses.v1.networking.k8s.io -v=6
  /apis/networking.k8s.io/v1/namespaces/default/ingresses
```

Or you can ask for the resource without qualification, and `kubectl` will do some discovery to figure out the api group and url.

```
kubectl -n default get ingress awesome -v=6
  /apis/networking.k8s.io/v1/namespaces/default/ingresses
```

Something is responsible for converting whatever's stored inside the database to the requested format,
and for converting submitted (eg with `kubectl apply -f`) resource bodies to the database format.
You don't do that.

All you need to do is stop accessing deprecated urls, and expecting to send/receive deprecated body formats.

That means update yaml in your git repos and wiki pages. And if you wrote code to interact with the kubernetes api-server, update your code.


## Outside Scope

If you were creating Custom Resource Definitions and writing a controller to manage them,
and deprecating a version of that CRD,
then _you or your controller_ would be responsible for doing the migration work.

But if you were, you probably wouldn't be reading this FAQ. See
https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/#upgrade-existing-objects-to-a-new-stored-version
