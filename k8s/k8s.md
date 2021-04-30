## k8s [Return Parent](../README.md)  

##### top

[TOC]





------



## 概念  
### KEP: 

Kubernetes Enhancement Proposals
Ref: https://github.com/kubernetes/enhancements/tree/master/keps

### Jobs:   

#### Parallel execution for Jobs

There are three main types of task suitable to run as a Job:

Non-parallel Jobs

- normally, only one Pod is started, unless the Pod fails.
- the Job is complete as soon as its Pod terminates successfully.

Parallel Jobs with a

fixed completion count:

- specify a non-zero positive value for `.spec.completions`.
- the Job represents the overall task, and is complete when there are `.spec.completions` successful Pods.
- when using `.spec.completionMode="Indexed"`, each Pod gets a different index in the range 0 to `.spec.completions-1`.

Parallel Jobs with a work queue:

- do not specify `.spec.completions`, default to `.spec.parallelism`.
- the Pods must coordinate amongst themselves or an external service to determine what each should work on. For example, a Pod might fetch a batch of up to N items from the work queue.
- each Pod is independently capable of determining whether or not all its peers are done, and thus that the entire Job is done.
- when *any* Pod from the Job terminates with success, no new Pods are created.
- once at least one Pod has terminated with success and all Pods are terminated, then the Job is completed with success.
- once any Pod has exited with success, no other Pod should still be doing any work for this task or writing any output. They should all be in the process of exiting.

For a *non-parallel* Job, you can leave both `.spec.completions` and `.spec.parallelism` unset. When both are unset, both are defaulted to 1.

For a *work queue* Job, you must leave `.spec.completions` unset, and set `.spec.parallelism` to a non-negative integer

#### parallelism

​	If .spec.parallelism is specified as 0, ***then the Job is effectively paused*** until it is increased.

​	Actual parallelism (number of pods running at any instant) may be more or less than requested parallelism, for a variety of reasons:

- For *work queue* Jobs, ***no new Pods are started after any Pod has succeeded*** -- remaining Pods are allowed to complete, however.
- The Job controller may throttle new Pod creation due to excessive previous pod failures in the same Job.
- When a Pod is gracefully shut down, it takes time to stop.

#### Completion mode  (Kubernetes v1.21 [alpha])

Jobs with *fixed completion count* - that is, jobs that have non null `.spec.completions` - can have a completion mode that is specified in `.spec.completionMode`:

- `NonIndexed` (default): the Job is considered complete when there have been `.spec.completions` successfully completed Pods. In other words, each Pod completion is homologous to each other. Note that Jobs that have null `.spec.completions` are implicitly `NonIndexed`.
- `Indexed`: the Pods of a Job get an associated completion index from 0 to `.spec.completions-1`, available in the annotation `batch.kubernetes.io/job-completion-index`. The Job is considered complete when there is one successfully completed Pod for each index. Note that, although rare, more than one Pod could be started for the same index, but only one of them will count towards the completion count.

#### Handling Pod and container failures

​		Note that even if you specify `.spec.parallelism = 1` and `.spec.completions = 1` and `.spec.template.spec.restartPolicy = "Never"`, the same program may sometimes be started twice.

#### Pod backoff failure policy

​		There are situations where you want to fail a Job after some amount of retries due to a logical error in configuration etc. To do so, set `.spec.backoffLimit` to specify the number of retries before considering a Job as failed.The back-off limit is set ***by default to 6***. Failed Pods associated with the Job are recreated by the Job controller with an exponential back-off delay (10s, 20s, 40s ...) capped at six minutes.

**Note:** If your job has `restartPolicy = "OnFailure"`, keep in mind that your container running ***the Job will be terminated once the job backoff limit has been reached.***

  [returntop](#top)



#### Job termination and cleanup                                                        

​		Once `.spec.backoffLimit` has been reached the Job will be marked as failed ***and any running Pods will be terminated.***

Another way to terminate a Job is by setting an active deadline. Do this by setting the `.spec.activeDeadlineSeconds` field of the Job to a number of seconds.Once a Job reaches `activeDeadlineSeconds`, ***all of its running Pods are terminated*** and the Job status will become `type: Failed` with `reason: DeadlineExceeded`.

Note that a Job's `.spec.activeDeadlineSeconds` takes precedence over its `.spec.backoffLimit`. Therefore, a Job that is retrying one or more failed Pods will not deploy additional Pods once it reaches the time limit specified by `activeDeadlineSeconds`, even if the `backoffLimit` is not yet reached.

#### Clean up finished jobs automatically

​		If the Jobs are managed directly by a higher level controller, ***such as [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/),*** the Jobs can be cleaned up by CronJobs based on the specified capacity-based cleanup policy.

Another way to clean up finished Jobs (either `Complete` or `Failed`) automatically is to use a ***TTL mechanism provided by a [TTL controller](https://kubernetes.io/docs/concepts/workloads/controllers/ttlafterfinished/)*** for finished resources, by specifying the `.spec.ttlSecondsAfterFinished` field of the Job.

#### Advanced usage

​		To suspend a Job, you can update the `.spec.suspend` field of the Job to true; later, when you want to resume it again, update it to false. 

**Note:** Suspending Jobs is available in Kubernetes versions 1.21 and above. You must enable the `SuspendJob` [feature gate](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/) on the [API server](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) and the [controller manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/) in order to use this feature.

When a Job is resumed from suspension, ***its `.status.startTime` field will be reset to the current time. This means that the `.spec.activeDeadlineSeconds` timer will be stopped and reset*** when a Job is suspended and resumed.

Remember that suspending a Job will ***delete all active Pods***. When the Job is suspended, ***your [Pods will be terminated](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination) with a SIGTERM signal.*** Pods terminated this way will not count towards the Job's `completions` count.

#### Specifying your own Pod selector

​		Normally, when you create a Job object, ***you do not specify `.spec.selector`.*** The system defaulting logic adds this field when the Job is created. It picks a selector value that will not overlap with any other jobs.

​		Say Job `old` is already running. You want existing Pods to keep running, but you want the rest of the Pods it creates to use a different pod template and for the Job to have a new name. You cannot update the Job because these fields are not updatable. Therefore, you delete Job `old` but *leave its pods running*, using `kubectl delete jobs/old --cascade=false`. Before deleting it, you make a note of what selector it uses: (这个应用场景有点意思)。

```
kubectl get job old -o yaml
```

```
kind: Job
metadata:
  name: old
  ...
spec:
  selector:
    matchLabels:
      controller-uid: a8f3d00d-c6d2-11e5-9f87-42010af00002
  ...
```

Then you create a new Job with name `new` and you explicitly specify the same selector. Since the existing Pods have label `controller-uid=a8f3d00d-c6d2-11e5-9f87-42010af00002`, they are controlled by Job `new` as well.

***You need to specify `manualSelector: true` in the new Job*** since you are not using the selector that the system normally generates for you automatically.

[returntop](#top)

## kubernetes架构
### 节点心跳机制

​		kubernetes节点心跳有两种形式：`NodeStatus`的更新 和 `Lease` 对象。lease是一种轻态资源， 

在大规模集群环境下，lease能够提升节点心跳的性能。

The kubelet is responsible for creating and updating the `NodeStatus` and a Lease object.

- The kubelet updates the `NodeStatus` either when there is change in status or if there has been no update for a configured interval. The default interval for `NodeStatus` updates is 5 minutes, which is much longer than the 40 second default timeout for unreachable nodes.
- The kubelet creates and then updates its Lease object every 10 seconds (the default update interval). Lease updates occur independently from the `NodeStatus` updates. If the Lease update fails, the kubelet retries with exponential backoff starting at 200 milliseconds and capped at 7 seconds.

##### 


