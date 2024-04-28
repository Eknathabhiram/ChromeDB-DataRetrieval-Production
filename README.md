# ChromeDB-Production-Deployment

ChromaDB is a robust open-source vector database that is highly versatile for various tasks such as information retrieval. This repository provides Kubernetes configuration files to facilitate the deployment of ChromaDB in a production environment. It's recommended to run ChromaDB in client/server mode for optimal performance, even though persistent client options are available.


## Running Locally
If you want to run ChromaDB on your local system, execute the following command:
```docker run --name chroma_container1 -p 8000:8000 chromadb/chroma:latest```

This command sets up ChromaDB with normal port mapping without any data persistence. If you need a custom port, use the following command:
'''docker run --name chroma_container1 -p 9000:9000 chromadb/chroma:latest --workers 1 --host 0.0.0.0 --port 9000 --proxy-headers --log-config chromadb/log_config.yml --timeout-keep-alive 30'''

## Getting Started
Assuming your production setup already has dynamic provisioning available, you will need to create a persistent volume to ensure your data survives even if your database shuts down.

### chromadb-pvc.yaml
This YAML file defines the PersistentVolumeClaim (PVC) for Chromadb, ensuring persistent storage for the database. Here's what it includes:

Metadata: Contains metadata about the PVC, including its name (name: chromadb-pvc) and labels (labels: app: "chroma-db").
Spec: Defines the PVC's specifications, such as access modes, storage class, and resource requests.
Access Modes: Specifies the access mode(s) for the volume (accessModes: ReadWriteOnce).
Storage Class: Specifies the storage class for the volume (storageClassName: rook-ceph-block).
Resources: Defines the requested storage capacity (storage: 2Gi).

Run: '''kubectl apply -f chromadb-pvc.yaml -n <your-namespace-name>

chroma-final consists 2 parts.
 1. chroma-svc.yaml
 2. chroma-sts.yaml

### chromadb-svc.yaml
This YAML file defines the Kubernetes Service for accessing Chromadb(basically for locating chromadb inside the cluster). Here's what it includes:

Metadata: Contains metadata about the service, including its name (name: chromadb-svc).
Spec: Defines the service's specifications, such as selector labels and ports.
Selector: Specifies the labels used to select the pods that the service will route traffic to (app: chromadb-prod).
Ports: Specifies the ports on which the service will listen for incoming traffic (port: 8000).

### chromadb-sts.yaml
We use statefulset for the deployment of databases because they retain the state of database...Even the pod is deleted or restarted they will revert back to their previous state.
i.e database structure like collection name ..user data etc. won't be lost.
This YAML file defines the Kubernetes StatefulSet for managing Chromadb deployment. Here's what it includes:

Metadata: Contains metadata about the StatefulSet, including its name (name: chromadb-dep).
Spec: Defines the StatefulSet's specifications, such as the service name it's associated with (serviceName: "chromadb-svc") and the number of replicas (replicas: 1).
Selector: Specifies the labels used to select the pods managed by the StatefulSet (matchLabels: app: chromadb-prod).
Template: Defines the pod template for the StatefulSet, including labels and pod specifications.
Volumes: Specifies the volumes mounted to the pods, such as the persistent volume claim (name: chroma-data).
Containers: Defines the container(s) running in the pods, including the image, ports, and volume mounts.

Run: '''kubectl apply -f chromadb-final.yaml -n <your-namespace-name>

## Extra Mandatoty Stuff:
Ensure to add liveness and readiness probes, as well as specify the resources provided for the database in the sts.yaml file. 
