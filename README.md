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
## API changes
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

## API-Server changes
For pods which has been created with new podSpec, it will set `PodStatus.ResourcesAllocated` field as same as PodSpec.Container.Resources.

## Scheduler level changes

## node level changes
### Kubelet
#### Pod admission stage
At pod admission stage(the moment before starting to create a pod), kubelet will see `PodStatus.ResourceAllocated` field not a PodSpec.Container.Resources.

#### Pod Resizing
Kubelet will accept resize request if there is enough room for newly requested resources. Sum of resources of pod(including resized resources) would not be greater than amount of allocatable resources.

Once request accepted, kubelet will update `PodStatus.ResourceAllocated` field as same as `PodSpec.Container.Resources` then update `Pod.Status.Resize[]` field to `InProgress`. At the last stage it will make CRI-API call for `UpdateContainerResources`.


### CRI
This feature introduces significant changes on CRI(at least for me).

First, it introduces platform agnostic ContainerResources message type for CRI and make UpdateContainerResource API to use it, which currently take LinuxContainerResources only. New ContaineResources type encapsulates both resource types for linux and windows.

Second, it provides a new machanism to parse current cgroup stats of container from CRI runtime(containerd/crio/etc) by adding ContainerResourse message type on ContainerStatus message.

### Runtimes

