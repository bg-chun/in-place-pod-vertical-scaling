# Background
## What this feature actually does?
There was no mechanism to update container's resource req/limit in place without re-deploying pod.
Since resizing resource req/limit means change limit of pod and container's cgroup.
The fundermantal idea behind this feature is to provide way to resize pod and container's cgroup if it is possible on the node where target pod/container has been bound.

## What is vertical scaling?
- TBD

## Who needs it?
- seems gke



# KEP & PR
- https://github.com/kubernetes/enhancements/commits/master/keps/sig-node/1287-in-place-update-pod-resources
- https://github.com/kubernetes/kubernetes/pull/102884

# Changes
## API level changes
This feature introduces following changes on PodSpec and PodStatus.
- PodSpec.Container.Resources becomes mutable for CPU and memory resource types.
- PodSpec.Container.ResizePolicy (new object) gives users control over how their containers are resized.
- PodStatus.Resize status describes the state of a requested Pod resize.
- PodStatus.ResourcesAllocated describes node resources allocated to Pod.
- PodStatus.Resources describes node resources applied to running containers by CRI.

### Container Resize Policy
`PodSpec.Container[].ResizePolicy` provides a way to describe preference of restarting. RestartNotRequired(Default) and Restart are supported.
But... RestartNotRequired does not gurantee that a container won't be restarted :(


### Resize Status
`Pod.Status.Resize[]` field shows that whether kubelet has accepted or rejected a resize request for a particular resource such as cpu and memory.
The possible values are Proposed/InProgress/Deferred(delayed)/Infeasible(rejected).


## Scheduler level changes

## node level changes
### Kubelet
### CRI
This feature introduces significant changes on CRI(at least for me).

First, it introduces platform agnostic ContainerResources message type for CRI and make UpdateContainerResource API to use it, which currently take LinuxContainerResources only. New ContaineResources type encapsulates both resource types for linux and windows.

Second, it provides a new machanism to parse current cgroup stats from CRI runtime(containerd/crio/etc) by adding ContainerResourse message type on ContainerStatus message.





### Runtimes

