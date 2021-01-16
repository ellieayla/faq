A Kubernetes CronJob may fail to start many times in a row, and then get stuck reporting:

Cannot determine if job needs to be started: Too many missed start time (> 100). Set or decrease .spec.startingDeadlineSeconds or check clock skew.

This should be based purely on the maths `inseconds(now - status.lastScheduleTime) / spec.startingDeadlineSeconds > 100` in https://github.com/kubernetes/kubernetes/blob/6d5cb36d36f34cb4f5735b6adcd5ea8ebb4440ba/pkg/controller/cronjob/utils.go#L95

###

```
kubectl get cronjob foo
...
status:
  lastScheduleTime: "2021-01-15T03:00:00Z"
```

## Can we change the CronJob's `status.lastScheduleTime` field?

Not with `kubectl patch`:
```
kubectl patch cronjob foo -p '{"status":{"lastScheduleTime":"2021-01-15T03:00:01Z"}}'
cronjob.batch/foo (no change)
```

## But we can patch the `/status` subresource directly:

```
kubectl proxy
Starting to serve on 127.0.0.1:8001
```

```
curl -XPATCH -H "Accept: application/json" -H "Content-Type: application/strategic-merge-patch+json" \
  http://127.0.0.1:8001/apis/batch/v1beta1/namespaces/default/cronjobs/foo/status \
  -d '{"status":{"lastScheduleTime":"2021-01-15T03:00:01Z"}}'
```

```
kubectl get cronjob foo -o go-template='{{.status}}'
map[lastScheduleTime:2021-01-15T03:00:01Z]%
```
