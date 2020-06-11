# Tetration as a Kubernetes Daemonset  

>WARNING: THIS IS UNSUPPORTED LAB CODE AND SHOULD NOT BE USED IN PRODUCTION

## Overview

This repo will guide you through integrating Tetration and Kubernetes via a Daemonset.

The steps taken are:
1. Create a user for the Tetration control plane integration
2. Create an external orchestrator using new Tetration users API token
3. Download and insert the Tetration Linux install script into a ConfigMap
4. Apply the ConfigMap and DaemonSet to the cluster

### Setting up Tetration user

Tetration learns about nodes, pods, and services through the Kubernetes API server

We will create an admin role for Tetration. Create a user with lower privileges in production.
```bash
$ kubectl apply -f rbac/01-authentication.yaml
```

Extract the API token for the Tetration user
```bash
$ kubectl apply -f rbac/01-authentication.yaml
```

Head to the Tetration interface to create a Kubernetes external orchestrator. Use the token received above.

> Don't forget to _uncheck_ "secure connector" 

### Copying the install script

The DaemonSet needs to use the Tetration agent installer script tied to your cluster.

On Tetration, navigate to "Cog > Agent Config > Software Agent Download", select the following options:

- Linux
- Enforcement Agent

Download the installer script and open in a text editor. Copy all of the text.

Open `agent/02-install-script.yaml`

Copy the install script text below the ConfigMap metadata

**It is _critical_ that the install script text is indented like below (4 spaces)**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: install-script
  namespace: tetration
data:
  install.sh: |
    # Update and install packages - keep this here
    apt-get update 
    apt-get install unzip rpm -y

    # PASTE INSTALLER SCRIPT HERE
    # KEEP FOUR (4) SPACE INDENT
    # REPLACE ALL TEXT BELOW THIS LINE
    # =============================================

    # This script requires privilege users to execute.
    #
    # If pre-check is not skipped, checks prerequisites for installing and running
    # tet-sensor on Linux hosts.
    #
    # If all prerequisites are met and installation succeeds, the script exits
    # with 0. Otherwise it terminates with a non-zero exit code for the first error
    # faced during execution.

    ... remaining text cut ...
```

### Applying the ConfigMap and DaemonSet

At this point we can now launch the Tetration agent

```bash
$ kubectl apply -f agent
```

### Checking the Tetration agent

You can check that the Tetration agent is deployed on all nodes:
```bash
$ kubectl get pods -n tetration
NAME          READY   STATUS    RESTARTS   AGE
agent-wscvj   1/1     Running   0          13d
```

You can check the status of the DaemonSet:
```bash
$ kubectl get daemonset -n tetration
NAME    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
agent   1         1         1       1            1           <none>          13d
```

### Further Notes

Cloud providers tested again:
- Azure AKS

To be tested:
- AWS EKS
- Google GKE