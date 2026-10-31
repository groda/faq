# kubectl

`kubectl` talks to a Kubernetes API server. It does not manage local containers by itself, and it does not read the cluster from the current directory unless you point it at a file. A kubeconfig holds the server URL, the credentials, and the current context. The current context picks the cluster and the user. The namespace flag picks the namespace. With no verb it prints help and exits non-zero. A verb that the API accepts exits 0. A rejected call prints the reason on standard error and exits non-zero. The upstream `kubectl` on Linux is assumed here. The same client runs on macOS. BusyBox does not include it. A name is not a label. A namespace is not a cluster.

Basic form: kubectl get pods

The verb comes first, then the resource, then names. Global flags such as `-n`, `--context`, and `-o` may sit before or after the verb. A name that starts with `-` is rare. `--` before a command belongs on `exec` and `logs` when the rest of the line is for the container. The shell still expands globs. Quote a label selector. `kubectl` does not ask before `delete`.

## List pods
Also asked as: kubectl get pods; list pods; get po; what is running; kubectl pods
`get pods` lists pods in the current namespace. The columns are name, ready, status, restarts, and age. `Ready` is `1/1` when every container is ready. `Running` can still be `0/1` if the process is up and the readiness probe is failing. `-o wide` adds the node and the pod IP. `-A` lists every namespace. `-n` picks one.

```sh
kubectl get pods -n namespace
```

People read `Running` as healthy. The ready count is the health the Service uses. People run `get pods` and see nothing because the context's namespace is `default` and the workload is elsewhere. `-A` is the check. A `Completed` pod is a finished Job, not a crashed one. A `CrashLoopBackOff` pod is still a row. `get pods` does not print the reason. `describe` does.

## Pick the cluster and the namespace
Also asked as: kubectl context; kubectl config current-context; kubeconfig; -n namespace; --context; wrong cluster
`config current-context` prints the context `kubectl` will use. `config get-contexts` lists them. `--context` overrides one command. `-n` overrides the namespace for one command. `config set-context --current --namespace` changes the default namespace. Credentials live in the kubeconfig. A context pointed at the wrong cluster is a quiet way to delete the wrong thing.

```sh
kubectl config current-context
```

People export nothing and assume the cluster. The file is `$KUBECONFIG`, or `~/.kube/config` if that is unset. `KUBECONFIG` can be a list of files. A 401 means the user in the context was rejected. A timeout means the server URL was not reached. Those are different. `get ns` is the small call that proves the context works. It does not prove you can create pods. `auth can-i` does.

## Show why a pod is not ready
Also asked as: kubectl describe pod; events; CrashLoopBackOff; ImagePullBackOff; pending pod; why is the pod failing
`describe pod` prints the spec, the container state, the last probe, and the events at the bottom. Events are the useful part. `FailedScheduling` is a node problem. `ImagePullBackOff` is the image name or the pull secret. `CrashLoopBackOff` is the process exiting. `describe` does not follow logs. It shows the last state the kubelet reported.

```sh
kubectl describe pod -- pod
```

People `describe` a name that is a Deployment. The pod name is the generated one from `get pods`. A describe of the Deployment shows replica status, not the container exit code. Events expire. An old failure can fall off the list. The container's last state is still on the pod. `describe` does not change the pod. It is safe to run. A missing name is a non-zero exit, not an empty describe.

## Read logs
Also asked as: kubectl logs; pod logs; logs -f; --previous; container logs; why did it crash
`logs` prints one container's stdout and stderr. `-f` follows. `--previous` prints the previous crashed container, which is the log you want after a restart. `-c` picks the container when the pod has more than one. `--tail` limits lines. The log is what the process wrote. It is not the event list. A pull failure has no container log.

```sh
kubectl logs --previous -c container -- pod
```

People follow logs on a CrashLoop and see nothing, because they are attached to the new empty stream. `--previous` is the exited one. A multi-container pod without `-c` errors and names the containers. `logs` does not include files the process wrote inside the container. Those are gone when the container is gone, unless a volume holds them. `-f` exits when the container exits, on a finished pod. It does not exit just because you deployed a new one.

## Run a command in a container
Also asked as: kubectl exec; shell into a pod; kubectl exec -it; exec bash; run a command in a container
`exec` runs one command inside an existing container. `-it` allocates a terminal, which a shell needs. `--` ends kubectl flags and starts the command. The container image has to contain the binary. A distroless image often has no shell. `exec` is not `docker exec` against the node. It goes through the API server and the kubelet.

```sh
kubectl exec -it -- pod -- sh
```

People write `kubectl exec pod bash` and kubectl reads `bash` as a flag or as a missing container form. The `--` is the separator. A pod with two containers needs `-c`. `exec` into a `CrashLoopBackOff` pod fails if no container is running. It does not start the pod. A shell you leave open is a process in the container. It is not a debug sidecar. `debug` is the newer verb for an ephemeral container, and the cluster has to allow it.

## Apply a manifest
Also asked as: kubectl apply; apply -f; declarative apply; kubectl apply file; three way merge
`apply -f` sends the manifest to the API and records last-applied state so a later apply can remove fields you deleted from the file. `-f` is a file, a directory, or `-` for standard input. A directory is applied in order, not as one transaction. `apply` creates the object if it is missing. It does not ask. `--dry-run=server` shows the result the server would store without storing it.

```sh
kubectl apply -f file
```

People edit a live object with `edit`, then apply an old file, and the apply puts the file back. The file is the source if you are using apply. People `apply` a Deployment and expect the pods to be gone and recreated immediately. The rollout is separate. `rollout status` waits for it. A YAML list of documents is one file. A missing namespace in the file lands in the current namespace, or fails if the file set one you cannot use.

## Delete a resource
Also asked as: kubectl delete; delete a pod; delete -f; remove a deployment; kubectl delete does not ask
`delete` removes the named object, or every object in a file. It does not ask. A deleted pod is recreated if a Deployment or ReplicaSet still wants it. Delete the owner to remove the workload. `--force --grace-period=0` skips the graceful kill. It is for a stuck pod, not for a normal deploy. `-l` deletes by selector. A typo in the selector can match more than you think.

```sh
kubectl delete -f file
```

People delete a pod to "restart" it and the new pod keeps the same bug, because the Pod template did not change. `rollout restart` rolls the workload. People `delete -f` the same file they applied and are surprised the namespace object in the file went too. Read the file. `delete` exits non-zero if the name is already gone, unless you pass `--ignore-not-found`. A foreground cascade waits for dependents. `--wait=false` returns before they are gone.

## Restart a workload
Also asked as: kubectl rollout restart; bounce pods; rolling restart; restart deployment; pick up a new config
`rollout restart` annotates the Pod template so a Deployment, StatefulSet, or DaemonSet starts a rollout. New pods come up. Old pods terminate as the strategy allows. It does not rebuild an image. The same image tag is pulled only if the node does not already have it, unless the image pull policy is `Always`. `rollout status` waits until the rollout finishes or you interrupt it.

```sh
kubectl rollout restart deployment -- deploy
```

People restart to pick up a ConfigMap change. Pods that mount a ConfigMap as a volume do not always see the new data until they are recreated. A restart does that. Pods that read the ConfigMap only at process start also need the restart. `rollout undo` returns the previous template. It does not undo a data migration the new pods already did. `rollout status` exiting non-zero means the rollout stalled or failed. `describe deployment` has the condition.

## Scale a workload
Also asked as: kubectl scale; change replicas; scale deployment; scale to zero; how many pods
`scale` sets the replica count on a Deployment, ReplicaSet, or StatefulSet. `--replicas=0` stops the pods and leaves the object. It is the off switch that is not a delete. `get deploy` shows desired, current, and ready. Ready catching up is a rollout. Scale does not change the image or the resources.

```sh
kubectl scale --replicas=3 deployment -- deploy
```

People scale a pod. A bare pod has no replica field. Scale the Deployment that owns it. People scale up and the new pods stay `Pending`. That is scheduling, not a failed scale. `describe pod` on a pending pod names the shortage. A HorizontalPodAutoscaler will overwrite your replica count if it is managing that object. `get hpa` is the check before you scale by hand.

## Port-forward to a pod
Also asked as: kubectl port-forward; forward a local port; reach a pod port; port-forward svc; localhost to cluster
`port-forward` listens on your machine and forwards to a port on one pod, or on a Service. `svc/name` picks a pod behind the Service. The forward dies when `kubectl` exits. It is not a firewall rule, and it is not visible to other users unless you bind a non-local address. The remote port is the container port, not the Service's node port.

```sh
kubectl port-forward svc/service 8080:80
```

People forward the Service port and the container is listening on another. The map is `local:remote`. A connection refused after a successful forward is the process inside the pod. A bind error is the local port. `port-forward` does not survive a pod replacement if you targeted a pod name. Targeting the Service follows a current ready pod, and it still drops when you stop the command.

## Change the output
Also asked as: kubectl -o yaml; -o json; -o name; -o wide; jsonpath; machine readable get
`-o yaml` and `-o json` print the API object. `-o name` prints `pod/name`, which is the form to pass back to another command. `-o wide` adds columns on `get`. `-o jsonpath=` pulls one field. These do not change the object. `--dry-run=client -o yaml` on `create` prints a manifest and does not send it. That is how you generate a file.

```sh
kubectl get pod -o name -n namespace
```

People parse the table. The columns move when `-o wide` is on, and custom columns differ by version. `-o name` and `-o json` are the stable ones. `jsonpath` wants the API field, which is often `.status.podIP`, not the table header. A missing field prints empty and exits 0. Check the yaml once. `-o yaml` on a list is a List object, not one document per pod, unless you asked for a single name.

## Select by label
Also asked as: kubectl -l; label selector; get pods by label; app=name; selector not name
`-l` filters a list or a delete by label. `app=api` is equality. `app in (api,web)` is a set. The selector is not a name substring. Labels are on the object. A Deployment's labels and its pod template labels can differ. `get pods -l` matches pod labels, not the Deployment name. Quote the selector so the shell does not split it.

```sh
kubectl get pods -l 'app=pattern' -n namespace
```

People label the Deployment and wonder why `get pods -l` is empty. The pods have the template labels. `get deploy -l` is the other query. A delete with a wrong selector deletes every match in the namespace. Print the names with `-o name` before you delete by selector. There is no confirmation. An empty match deletes nothing and exits 0. That looks like success. It is an empty set.

## Explain a field
Also asked as: kubectl explain; api field; explain pod spec; what does this field mean; openapi explain
`explain` prints the API documentation for a resource or a dotted field. `explain pod.spec.containers` is the container list. It does not read your cluster's live objects. It reads the API discovery from the server, so the text matches the version you are pointed at. It is the field name `jsonpath` wants.

```sh
kubectl explain pod.spec.containers
```

People search the manifest for a word and guess the path. `explain` is the path. A field that is not in your server's version is absent, not hidden. A CRD explains only if the CRD published OpenAPI. `explain` does not validate a file. `apply --dry-run=server` is the validation. `explain` exiting non-zero means the resource or the field is unknown on this cluster.

## See if you are allowed
Also asked as: kubectl auth can-i; rbac; forbidden; can I create a pod; permission denied
`auth can-i` asks the API if this user can perform a verb on a resource. `auth can-i create deployments -n namespace` is the question. `--list` dumps what you can do. A 403 on a real call is the same check failing for real. `can-i` does not grant anything. It does not switch users. The user is the one in the current context.

```sh
kubectl auth can-i create deployments -n namespace
```

People see `yes` in `default` and `no` in the namespace they deploy to. The namespace is part of the question. A `yes` on `create` is not a `yes` on `delete`. Impersonation is `--as`, and the cluster has to allow you to impersonate. A forbidden from `apply` names the resource it could not write. That resource may be in the file under a different kind than the one you checked.

## Roll back a deployment
Also asked as: kubectl rollout undo; rollback; rollout history; previous revision; bad deploy
`rollout undo` returns a Deployment to the previous revision, or to `--to-revision`. `rollout history` lists revisions. Undo applies the stored Pod template. It does not undeploy a migration, and it does not put back a ConfigMap you overwrote unless that ConfigMap is still the old object. `rollout status` waits for the undo the same way it waits for a deploy.

```sh
kubectl rollout undo deployment -- deploy
```

People undo and the pods keep the new behavior because the image tag was `latest` and the old revision also says `latest`. The tag moved. Pin an image digest if undo must mean the same bytes. History is trimmed. A revision that aged out cannot be undone to. `rollout pause` stops a Deployment from rolling. Forgetting `rollout resume` looks like a stuck deploy. `status` tells you it is paused.

## What a failed kubectl means
Also asked as: kubectl exit code; kubectl failed; connection refused; forbidden exit status; context deadline
`kubectl` exits 0 when the API call did what you asked. It exits non-zero on a bad flag, a missing object, a 403, or a timeout. The message is on stderr. A timeout is often a network path or an API server, not a wrong namespace. A forbidden is policy. A not-found is the name or the namespace. `--request-timeout` bounds the wait. It does not retry the way a controller does.

```sh
kubectl get pods -n namespace
```

People script `get` and treat empty output as failure. An empty namespace exits 0 and prints a header or nothing, depending on `-o`. Test the status, then the rows. `delete` of a missing name is non-zero unless `--ignore-not-found` is set. Under `set -e` that aborts a cleanup. A dry run exits 0 when the server accepted the preview. It did not persist the object. The next real apply is the one that matters.
