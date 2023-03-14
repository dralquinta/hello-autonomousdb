# Scalein and Scaleout process using CronJobs

As this is a scheduled task, I've created two yaml that allow to scale in and scale out a deployment by specific time in the day. 

`hpa-scaleout.yaml`

The CronJob runs every Monday to Friday at 12:13 PM (schedule: "30 13 * * 1-5") and executes a kubectl command to scale the "samplespringboot-deployment" to the desired number of replicas in the "samplespringboot" namespace (-n samplespringboot).

Replace <image-name> with the name of the Docker image that contains the kubectl command, and <desired-replicas> with the desired number of replicas.

`hpa-scalein.yaml`

The CronJob runs every Monday to Friday at 12:16 PM (schedule: "35 14 * * 1-5") and executes a kubectl command to scale the "samplespringboot-deployment" to one replica in the "samplespringboot" namespace (-n samplespringboot).

Replace <image-name> with the name of the Docker image that contains the kubectl command.


This uses dockerfile already containing CLI. Check the specs of the file for further details. 

Docker file requires: 

- kubeconfig file
- ociconfig file
- private API key

Upon execution to push into kubernetes cluster, this is how it looks: 

```shell
./build-tag-release-kubectl-executor.sh kubectl-executor oracleidentitycloudservice/denny.alquinta@oracle.com sa-santiago-1 idlhjo6dp3bd
Enter OCIR Token
YOUR_TOKEN_HERE
--- login into OCIR ---
WARNING! Your password will be stored unencrypted in /home/ubuntu/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
--- pruning images ---
Deleted Containers:
e9e770b7d61609695aac090dc44860e66219d0d44ba68e151f1b26fc39f77d7f
614ade3bc06f008722431705afa3a35403eb422fd8393588f15403dc34dce057
...
...
...

deleted: sha256:fbb641a8b94349e89886f65d79928e4673530e2a2b4d33c2c95e0426713f78e4

Total reclaimed space: 1.727GB
--- DEBUG HERE building image kubectl-executor:latest ---
Sending build context to Docker daemon  20.48kB
Step 1/11 : FROM jpoon/oci-cli:latest
latest: Pulling from jpoon/oci-cli
e79bb959ec00: Pull complete 
d4b7902036fe: Pull complete 
1b2a72d4e030: Pull complete 
d54db43011fd: Pull complete 
69d473365bb3: Pull complete 
7dc3a6a0e509: Pull complete 
a288a79001c3: Pull complete 
7d3cdae56021: Pull complete 
dbf17696f820: Pull complete 
55edb98cef4c: Pull complete 
f8d5fb2bc38c: Pull complete 
db01ea420331: Pull complete 
a584ad583f88: Pull complete 
Digest: sha256:07424a75408952170dcc897bc9ecb4458c8ec7ac2f25262e45a8b420ac77117f
Status: Downloaded newer image for jpoon/oci-cli:latest
 ---> 80001798b953
Step 2/11 : RUN apt-get update &&     apt-get install -y curl gnupg2 wget ca-certificates python3 tar gzip bash &&     rm -rf /var/lib/apt/lists/*
 ---> Running in c1da8919bc8e
Ign:1 http://deb.debian.org/debian stretch InRelease
Get:2 http://deb.debian.org/debian stretch-updates InRelease [93.6 kB]
Get:3 http://deb.debian.org/debian stretch Release [118 kB]
Get:4 http://security.debian.org/debian-security stretch/updates InRelease [59.1 kB]
Get:5 http://deb.debian.org/debian stretch Release.gpg [3177 B]
Get:6 http://security.debian.org/debian-security stretch/updates/main amd64 Packages [782 kB]
...
...
...
Updating certificates in /etc/ssl/certs...
15 added, 29 removed; done.
Setting up libcurl4-openssl-dev:amd64 (7.52.1-5+deb9u16) ...
Setting up curl (7.52.1-5+deb9u16) ...
Processing triggers for ca-certificates (20200601~deb9u2) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
Removing intermediate container c1da8919bc8e
 ---> eb8a850fa2b8
Step 3/11 : ENV KUBE_LATEST_VERSION="v1.21.5"
 ---> Running in beaff6ff86a3
Removing intermediate container beaff6ff86a3
 ---> 77b46a62575e
Step 4/11 : RUN curl -L https://storage.googleapis.com/kubernetes-release/release/$KUBE_LATEST_VERSION/bin/linux/amd64/kubectl -o /usr/local/bin/kubectl     && chmod +x /usr/local/bin/kubectl
 ---> Running in cd078742ae53
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 44.4M  100 44.4M    0     0  20.4M      0  0:00:02  0:00:02 --:--:-- 20.4M
Removing intermediate container cd078742ae53
 ---> 06d37fdb77c0
Step 5/11 : RUN mkdir -p /root/.kube
 ---> Running in 5445a54b9362
Removing intermediate container 5445a54b9362
 ---> 9f07e776bd37
Step 6/11 : COPY kubeconfig /root/.kube/config
 ---> b47bfaf31831
Step 7/11 : RUN mkdir -p /root/.oci
 ---> Running in 1da82dd90a37
Removing intermediate container 1da82dd90a37
 ---> 7b4ccf92e886
Step 8/11 : COPY ociconfig /root/.oci/config
 ---> f18d9c19eee1
Step 9/11 : COPY private /root/.oci/private
 ---> 794289525ead
Step 10/11 : RUN chmod 600 /root/.oci/private
 ---> Running in 9a963775bc73
Removing intermediate container 9a963775bc73
 ---> 0cf481707db6
Step 11/11 : ENTRYPOINT ["kubectl"]
 ---> Running in 6e487a77936b
Removing intermediate container 6e487a77936b
 ---> 555f013d26df
Successfully built 555f013d26df
Successfully tagged kubectl-executor:latest
--- tag image kubectl-executor:latest ---
--- push image kubectl-executor:latest ---
The push refers to repository [sa-santiago-1.ocir.io/idlhjo6dp3bd/kubectl-executor]
e4c143d9fabc: Pushed 
ae5ffd1a1e31: Pushed 
cbd2af299f45: Pushed 
f6312091aabf: Pushed 
716bd6fb12e4: Pushed 
f6d4262ec55d: Pushed 
bf14fb87f598: Pushed 
58e372c272f3: Pushed 
3822c3801f96: Pushed 
dcd3bfcea6e3: Pushed 
23b453a25a0e: Pushed 
b5d9f73a8fe7: Pushed 
b24438cea2cc: Pushed 
c76a8d0f7f24: Pushed 
b4074dfd8a23: Pushed 
9f72e27f9aa4: Pushed 
0fe19df8b8f8: Pushed 
b17cc31e431b: Pushed 
12cb127eee44: Pushed 
604829a174eb: Pushed 
fbb641a8b943: Pushed 
latest: digest: sha256:6a88d486a096d11be83e5761c99fbf80a7014a6ced587154402b444fb28361a8 size: 4727
--- all set! ---

```