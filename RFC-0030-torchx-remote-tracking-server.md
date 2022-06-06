# TorchX Remote Tracking Server

**Authors:**
* @diptanu


## **Summary**
TorchX has a tracking module which currently lets users save state in the form of key value pair to file system. The proposal here is to create a tracking service to track and organize experiments, allow logging arbritary data from the experiments and gather and visualize experiment metrics, and also operational metrics, such as resource utilazaion of training hardware and logs from job schedulers related to experiments.


## **Motivation**
### What motivates this proposal and why is it important?
There are various experiment managers for managing experiments, frameworks usually have an API to interface with them. The motivation is to 1. Develop standard TorchX interfaces to integrate with experiment managers 2. Provide a default open source experiment manager that works well with TorchX.

### How should users and developers think about this feature, how would it impact the way PyTorch is used?
Existing TorchX users will simply have the option for using a remote tracker for experiment management with this feature enabled. This will provide better availability of the experiment results over the existing file system backed storage. In addition there will be an API based access to the experiment data.

### Explain impact and value of this feature
* This feature extends the ResultTracker API to support some ideas(namespace, project, experiments,etc) which popular experiment tracking systems already support. So integration of experiment managers with torchx will be easier.
* Also we provide a default open source implementation of a tracker with pluggable storage, search, backends, etc, which users can use out of the box with torchx. 

## **Proposed Implementation**
There will be two components to this - 1. Client side interfaces 2. A remote tracking service
### Client Interfaces in torchx.tracking
We can re-use the existing tracking ResultTracker interface to log key value pairs, with some additional enhancements to set file type which allows users to log images or other files during training time.
The key will be namespaced based on the delimeter `/`. So, for ex, `langtech/asr/en_wav2vec/<exp_id>` will create a new experiment in the `langtech->asr->en_wav2vec` namespace. All the metrics, files and other artifacts can be logged via the ResultTracker api.

A new implementation of ResultTracker has to be written which is based on the RPC api of the remote tracker service.

### ACLs
Sensitive experiments could be ACLed away from public visibility inside the organization. We can use an existing RBAC library to implement such ACLs for the namespaces and other obects created in the tracker.

### Remote Tracking Service
The remote tracking service will use a grpc API, and a plugable storage backend. The first two implementations of the backend are going to be ephemeral without any external storage dependencies, and the second will be a SQL based storage. 
We will also have a plugable blob storage dependency to store arbitrary files during the experiment. Checkpoint and blob storage will not be in the scope of the tracker service, and external model stores are reccomended.

### Training Metrics
We will integrate with TorchMetrics and track training time metrics from the application. For values of keys which are subclassed from Metric we can automatically track them as Metrics and we can use appropriate visualization mechanism for specific meteics like accuracy. 

### Operational Metrics
We can expose hardware performance counters and such to the tracker. We obviously don't want to build a full fledged metrics store, so we could write them to metrics sinks, such as time series databases or to other cloud metrics services such as CloudWatch, DataDog, etc. In most cases there are already metrics services being used by users, the tracker could have pluggable implementations to add annotations and such to the metrics services with relevant information of experiments. So for example, we could annotate a time series chart with start and end time of an experiment.

## **Unresolved questions**
Many details about this proposal is un-resolved like the API on the tracking interface and such, but I am hoping we can work through them on PRs. Also, we can add more details here based on the conversation with maintainers of torchx.

### Next Steps
Once we get some concencus around building a remote tracker service for torchx, we can flesh out the design in PRs while building the API.
