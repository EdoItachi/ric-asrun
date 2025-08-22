# Near-RT RIC “j-release” — As-Run Report

* **Author:** Shyama7004  
* **Environment:** Single-node VM (containerd + kubeadm)  
* **KUBECONFIG:** `/etc/kubernetes/admin.conf`  
* **Branches:** `ric-plt-ric-dep@j-release`, `ric-plt-appmgr@j-release`  
* **Registry:** prefer `:10004`, fallback `:10002`  
* **Proxy:** `NO_PROXY` covers cluster and nexus  

---

## 1. Collected Outputs

All raw outputs are saved under [`/outputs`](./outputs):  

- [pods-all.txt](https://github.com/EdoItachi/ric-asrun/blob/main/outputs/pods-all.txt) — `kubectl get pods -A -o wide`  
- [svc-all.txt](https://github.com/EdoItachi/ric-asrun/blob/main/outputs/svc-all.txt) — `kubectl get svc -A`  
- [helm-list.txt](https://github.com/EdoItachi/ric-asrun/blob/main/outputs/helm-list.txt) — `helm list -A`  
- [deploy-hw-go.yaml](https://github.com/EdoItachi/ric-asrun/blob/main/outputs/deploy-hw-go.yaml) — hw-go Deployment manifest  
- [events-ricxapp.txt](https://github.com/EdoItachi/ric-asrun/blob/main/outputs/events-ricxapp.txt) — rollout events for hw-go  
- [health-ready-expected.txt](https://github.com/EdoItachi/ric-asrun/blob/main/outputs/health-ready-expected.txt) — ingress health check (`200`)  

---

## Steps, Issues, and Fixes

### Step-1: Did a quick sanity check

<details>
<summary>Commands Ran</summary>
    
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
sudo -E env KUBECONFIG=$KUBECONFIG kubectl version --output=yaml
sudo -E env KUBECONFIG=$KUBECONFIG kubectl get ns
sudo -E env KUBECONFIG=$KUBECONFIG helm list -A
sudo -E env KUBECONFIG=$KUBECONFIG kubectl get pods -A -o wide
````

</details>

<details>
    <summary>Outputs</summary>
<img width="977" height="549" alt="Screenshot 2025-08-21 at 18 30 55" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/1.1.png" />
<img width="983" height="540" alt="Screenshot 2025-08-21 at 18 31 58" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/1.2.png" />

</details>



### Step-2: Got deployment scripts and installed base bits

<details>
<summary>Commands Ran</summary>

```bash
mkdir -p ~/workspace && cd ~/workspace
[ -d ric-plt-ric-dep ] || git clone https://github.com/o-ran-sc/ric-plt-ric-dep.git
cd ric-plt-ric-dep
git checkout j-release
```

```bash
cd ~/workspace/ric-plt-ric-dep/bin
sudo ./install_k8s_and_helm.sh
sudo ./install_common_templates_to_helm.sh
```
</details>

<details>
    <summary>Outputs</summary>

From `sudo ./install_k8s_and_helm.sh`:

```bash
Done with master node setup
+ [[ ! -z '' ]]
+ [[ ! -z '' ]]
+ [[ ! -z '' ]]
+ [[ 1 -gt 100 ]]
+ [[ 1 -gt 100 ]]
```

From `sudo ./install_common_templates_to_helm.sh`:

```bash
servcm up and running
/root/.cache/helm/repository
Successfully packaged chart and saved it to: /tmp/ric-common-3.3.2.tgz
"local" has been removed from your repositories
"local" has been added to your repositories
checking that ric-common templates were added
NAME            	CHART VERSION	APP VERSION	DESCRIPTION                                   
local/ric-common	3.3.2        	           	Common templates for inclusion in other charts
```

<img width="979" height="132" alt="Screenshot 2025-08-21 at 18 32 40" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/2.png" />

</details>


### Step-3: Selected recipe and set IPs

<details>
<summary>Commands Ran</summary>

```bash
cd ~/workspace/ric-dep
# Choose the j-release recipe you used; example:
ls -l RECIPE_EXAMPLE
grep -nA2 -E 'extsvcplt:|ricip|auxip' RECIPE_EXAMPLE/example_recipe_oran_j_release.yaml
```

</details>

<details>
<summary>Outputs</summary>

Output from: `./install_k8s_and_helm.sh`

```bash
Done with master node setup
+ [[ ! -z '' ]]
+ [[ ! -z '' ]]
+ [[ ! -z '' ]]
+ [[ 1 -gt 100 ]]
+ [[ 1 -gt 100 ]]

````

Output from: `./install_k8s_and_helm.sh`
<img width="832" height="316" alt="Screenshot 2025-08-21 at 18 39 55" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/3.png" />

</details>

> recipes requires `extsvcplt.ricip` and `extsvcplt.auxip`.&#x20;


### Step-4: Deployed the RIC platform

<details>
<summary>Commands Ran</summary>

```bash
cd ~/workspace/ric-dep/bin
./install -f ../RECIPE_EXAMPLE/example_recipe_oran_j_release.yaml
```
</details>

<details>
  <summary>Ouputs</summary>

```bash

namespace/ricplt created
namespace/ricinfra created
namespace/ricxapp created
Deploying RIC infra components [infrastructure dbaas appmgr rtmgr e2mgr e2term a1mediator submgr vespamgr o1mediator alarmmanager ]
Note that the following optional components are NOT being deployed: {influxdb jaegeradapter}. To deploy them add them with -c to the default component list of the install command
configmap/ricplt-recipe created
Add cluster roles
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 7 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-infrastructure
LAST DEPLOYED: Thu Aug 21 15:21:36 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-dbaas
LAST DEPLOYED: Thu Aug 21 15:21:49 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-appmgr
LAST DEPLOYED: Thu Aug 21 15:21:58 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-rtmgr
LAST DEPLOYED: Thu Aug 21 15:22:08 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-e2mgr
LAST DEPLOYED: Thu Aug 21 15:22:17 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-e2term
LAST DEPLOYED: Thu Aug 21 15:22:26 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-a1mediator
LAST DEPLOYED: Thu Aug 21 15:22:35 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-submgr
LAST DEPLOYED: Thu Aug 21 15:22:45 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-vespamgr
LAST DEPLOYED: Thu Aug 21 15:22:54 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-o1mediator
LAST DEPLOYED: Thu Aug 21 15:23:02 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading ric-common from repo http://127.0.0.1:8879/charts
Deleting outdated charts
NAME: r4-alarmmanager
LAST DEPLOYED: Thu Aug 21 15:23:12 2025
NAMESPACE: ricplt
STATUS: deployed
REVISION: 1
TEST SUITE: None

```

  </details>

<details>
    <summary>Commands Ran</summary>

```bash
sudo -E env KUBECONFIG=$KUBECONFIG helm list -A
sudo -E env KUBECONFIG=$KUBECONFIG kubectl get pods -n ricplt -o wide
sudo -E env KUBECONFIG=$KUBECONFIG kubectl get pods -n ricinfra -o wide
```
</details>

<details>
    <summary>Outputs</summary>
<img width="977" height="538" alt="Screenshot 2025-08-21 at 18 33 27" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/4.png" />

</details>

<details>
<summary>Issues and fixes</summary>

**Issue**  
`submgr` and `vespamgr` were stuck in `ImagePullBackOff` / `ErrImagePull` because the default registry `:10004` was unreachable.

**Fix**  
Pulled the images from `:10002` and patched both Deployments:

```bash
sudo ctr -n k8s.io images pull nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-submgr:0.10.2
sudo ctr -n k8s.io images pull nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-vespamgr:0.7.5
kubectl -n ricplt set image deploy/deployment-ricplt-submgr  \
  container-ricplt-submgr=nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-submgr:0.10.2
kubectl -n ricplt set image deploy/deployment-ricplt-vespamgr \
  container-ricplt-vespamgr=nexus3.o-ran-sc.org:10002/o-ran-sc/ric-plt-vespamgr:0.7.5
kubectl -n ricplt patch deploy/deployment-ricplt-submgr  -p '{"spec":{"template":{"spec":{"containers":[{"name":"container-ricplt-submgr","imagePullPolicy":"IfNotPresent"}]}}}}'
kubectl -n ricplt patch deploy/deployment-ricplt-vespamgr -p '{"spec":{"template":{"spec":{"containers":[{"name":"container-ricplt-vespamgr","imagePullPolicy":"IfNotPresent"}]}}}}'
Both pods are now Running.

```
</details> 

### Step-5: Set recipe IPs (ricip and auxip)

<details>
<summary>Commands Ran</summary>

```bash
cd ~/workspace/ric-plt-ric-dep
nano RECIPE_EXAMPLE/example_recipe_oran_j_release.yaml
``` 
</details>

### Step-6: Verify ingress and AppMgr health
<img width="540" height="256" alt="Screenshot 2025-08-21 at 19 12 06" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/6.1.png" />

<details>
<summary>Commands Ran</summary>

```bash
# Find Kong proxy service and NodePort
sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt get svc | egrep -i 'kong|appmgr|proxy'
NODE_IP=$(hostname -I | awk '{print $1}')
curl -v "http://$NODE_IP:32080/appmgr/ric/v1/health/ready"
```
</details>

<details>
    <summary>Outputs</summary>
<img width="390" height="48" alt="Screenshot 2025-08-21 at 19 13 26" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/6.2.png" />

</details>

### Step-7: Fixed image pulls and ingress routing

#### first confirmed Kong service and NodePorts

<details>
    <summary>Commands Ran</summary>

```bash
sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt get svc | egrep -i 'kong|appmgr|proxy'
HTTP_NODEPORT=$(sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt get svc r4-infrastructure-kong-proxy \
  -o jsonpath='{.spec.ports[?(@.port==80)].nodePort}')
HTTPS_NODEPORT=$(sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt get svc r4-infrastructure-kong-proxy \
  -o jsonpath='{.spec.ports[?(@.port==443)].nodePort}')
echo "HTTP_NODEPORT=$HTTP_NODEPORT  HTTPS_NODEPORT=$HTTPS_NODEPORT"
````
</details>

<details>
    <summary>Output</summary>

<img width="764" height="173" alt="Screenshot 2025-08-21 at 19 12 59" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/7.1.png" />
</details>

#### created a minimal Kong Ingress for AppMgr

> Kong was installed, but I got a 404 without a matching route. I created one to map `/appmgr` to the AppMgr service and strip the prefix.

<details>
    <summary>Commands Ran</summary>

```bash
cat <<'EOF' >/tmp/appmgr-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: appmgr-ing
  namespace: ricplt
  annotations:
    konghq.com/strip-path: "true"
spec:
  ingressClassName: kong
  rules:
  - http:
      paths:
      - path: /appmgr
        pathType: Prefix
        backend:
          service:
            name: service-ricplt-appmgr-http
            port:
              number: 8080
EOF

sudo -E env KUBECONFIG=$KUBECONFIG kubectl apply -f /tmp/appmgr-ingress.yaml
```
</details>

<details>
    <summary>Outputs</summary>

<img width="560" height="77" alt="Screenshot 2025-08-21 at 19 08 58" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/7.2.png" />

</details>

<details>
<summary>Ingress check command</summary>

```bash
sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt get ingress -o wide
```
</details>

<details>
    <summary>Outputs</summary>

<img width="481" height="84" alt="Screenshot 2025-08-21 at 19 11 22" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/7.3.png" />

</details>

#### checked health through Kong

<details>
    <summary>Commands Ran</summary>

```bash
NODE_IP=$(hostname -I | awk '{print $1}')
curl -s -o /dev/null -w "%{http_code}\n" http://$NODE_IP:$HTTP_NODEPORT/appmgr/ric/v1/health/ready
```
</details>

<details>
    <summary>Outputs</summary>

<img width="628" height="92" alt="Screenshot 2025-08-21 at 19 11 01" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/7.4.png" />
</details>


> Sometimes I still got `404`. I waited \~30s and retried. If it kept failing, I ran:
>
> ```bash
> sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt describe ingress appmgr-ing | sed -n '/Rules:/,$p'
> sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt logs deploy/r4-infrastructure-kong --tail=100
> ```

#### Prove the image-pull fixes (submgr/vespamgr)

> I switched to `:10002` and set `IfNotPresent`.  

<details>
<summary>Commands Ran</summary>

```bash
for d in deployment-ricplt-submgr deployment-ricplt-vespamgr; do
  echo "== $d =="
  sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt get deploy $d \
    -o jsonpath='{range .spec.template.spec.containers[*]}{.name}{"  "}{.image}{"  "}{.imagePullPolicy}{"\n"}{end}'; echo
done

sudo ctr -n k8s.io images ls | egrep 'ric-plt-(submgr|vespamgr)'

sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricplt get pods -o wide | egrep 'submgr|vespamgr'
````

</details>

<details>
<summary>Outputs</summary>

<img width="942" height="177" alt="Screenshot 2025-08-21 at 19 10 41" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/7.5.png" />

<img width="987" height="147" alt="Screenshot 2025-08-21 at 19 10 13" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/7.6.png" />

<img width="980" height="168" alt="Screenshot 2025-08-21 at 19 08 28" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/7.7.png" />
</details>

#### What I got

* Kong proxy NodePorts found (HTTP and HTTPS)
* Ingress for `/appmgr` exists and health check returns **200**
* `submgr` and `vespamgr` are running, using images from `:10002` with `imagePullPolicy=IfNotPresent`


### Step-8: Ran ChartMuseum with Docker

<details>
<summary>Commands Ran</summary>

```bash
sudo docker run -d --name chartmuseum \
  -p 8080:8080 \
  -e DEBUG=1 \
  -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  chartmuseum/chartmuseum:latest

sudo docker ps | grep chartmuseum
sudo curl -s http://localhost:8080/health

export CHART_REPO_URL=http://localhost:8080
echo $CHART_REPO_URL
````

</details>

<details>
<summary>Outputs</summary>

<img width="921" height="68" alt="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/8.png" />
</details>

<details>
<summary>Issues and fixes</summary>

**Issue**  
`docker pull chartmuseum/chartmuseum:latest` failed — “Temporary failure in name resolution”.

**Fix**  
Fixed host/Docker DNS (systemd-resolved restart) and re-ran:

```bash
sudo docker run -d --name chartmuseum \
  -p 8080:8080 -e DEBUG=1 -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  chartmuseum/chartmuseum:latest
curl -s http://localhost:8080/health   # {"healthy":true}
ChartMuseum is healthy at http://localhost:8080.
```

</details> 

### Step-9: Installed the xApp onboarder (dms_cli)

#### installed Python 3.9 and cloned repo

<details>
<summary>Commands Ran</summary>

```bash
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.9 python3.9-venv
python3.9 --version

mkdir -p ~/workspace && cd ~/workspace
[ -d ric-plt-appmgr ] || git clone https://github.com/o-ran-sc/ric-plt-appmgr.git
cd ric-plt-appmgr && git checkout j-release
````

</details>

<details>
<summary>Outputs</summary>

<img width="407" height="201" alt="Screenshot 2025-08-21 at 19 40 57" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/9.1.png" />
</details>


#### created venv and installed onboarder CLI

<details>
<summary>Commands Ran</summary>

```bash
cd ~/workspace/ric-plt-appmgr/xapp_orchestrater/dev/xapp_onboarder
python3.9 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install .
dms_cli --help | head -n 5
```

</details>

<details>
<summary>Outputs</summary>

<img width="518" height="194" alt="Screenshot 2025-08-21 at 19 40 17" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/9.2.png" />
</details>


#### confirmed ChartMuseum is reachable (Docker path)

<details>
<summary>Commands Ran</summary>

```bash
export CHART_REPO_URL=http://localhost:8080
curl -s $CHART_REPO_URL/health        # {"healthy":true}
curl -s $CHART_REPO_URL/api/charts    # {} or []
```

</details>

<details>
<summary>Outputs</summary>

<img width="154" height="59" alt="Screenshot 2025-08-21 at 19 39 36" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/9.3.png" />
</details>

<details>
<summary>Issues and fixes</summary>

**Issue**  
`dms_cli get_charts_list --chart_repo_url` isn’t supported in this j-release.

**Fix**  
Relied on environment variable and basic health check instead:

```bash
export CHART_REPO_URL=http://localhost:8080
dms_cli health        # confirms CLI is ready
curl -s $CHART_REPO_URL/api/charts   # {} or []
Onboarder works with the local ChartMuseum.
```

</details> 

**Notes**

* In this j-release, `dms_cli get_charts_list` doesn’t take a `--chart_repo_url` flag. We’ll pass repo info directly when onboarding in Step 9.
* Success criteria: Python 3.9 installed, `dms_cli` works (`--help`), ChartMuseum health returns `{"healthy":true}`, and chart list is empty `{}` or `[]`.


### Step-10: Onboard and Verify hw-go xApp

<details>
<summary>Commands Ran</summary>

```bash
# Onboard the sample xApp
cd ~/workspace/ric-plt-appmgr/xapp_orchestrater/dev/xapp_onboarder
sudo -E env KUBECONFIG=$KUBECONFIG ./onboarder onboard config.json

# Install via Helm
sudo -E env KUBECONFIG=$KUBECONFIG helm install hw-go ./hw-go --namespace ricxapp

# Check pods
sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricxapp get pods -o wide
````

</details>

<details>
<summary>Outputs</summary>

```bash
NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE
ricxapp-hw-go-6bd8ff7d6f-67hq5   1/1     Running   0          2m    10.244.x.xxx   shyama7004
```

</details>

<details>
<summary>Issues and fixes</summary>

**Symptom**  
`Error from server (NotFound): deployments.apps "hw-go" not found`

**Cause**  
Helm release is `hw-go`, but the Deployment name is `ricxapp-hw-go`.

**Fix**
```bash
DEPLOY=$(kubectl -n ricxapp get deploy -o name | grep hw-go | head -n1 | cut -d/ -f2)
echo "Using DEPLOY=$DEPLOY"
kubectl -n ricxapp rollout status deploy/$DEPLOY --timeout=3m
```
</details>

## Step-11: Fix ErrImagePull & Stabilize Deployment

<details>
<summary>Actions Taken</summary>

1. Checked Deployment image and ReplicaSets.
2. Found pods pulling wrong image from `:10004`.
3. Patched Deployment to use correct registry `:10002`.
4. Scaled down and deleted bad ReplicaSets/pods.
5. Confirmed only one healthy ReplicaSet and pod.

</details>

<details>
<summary>Commands Ran</summary>

```bash
# Check Deployment image
sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricxapp get deploy ricxapp-hw-go \
  -o jsonpath='replicas={.spec.replicas} image={.spec.template.spec.containers[0].image}{"\n"}'

# Show ReplicaSets
sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricxapp get rs \
  -o jsonpath='{range .items[*]}{.metadata.name}{"  "}{.spec.template.spec.containers[0].image}{"  "}replicas={.spec.replicas} ready={.status.readyReplicas}{"\n"}{end}'

# Show pods
sudo -E env KUBECONFIG=$KUBECONFIG kubectl -n ricxapp get pods -o wide

# Confirm Helm release
sudo -E env KUBECONFIG=$KUBECONFIG helm list -n ricxapp
```

</details>

<details>
<summary>Outputs</summary>

```bash
replicas=1 image=nexus3.o-ran-sc.org:10002/o-ran-sc/ric-app-hw-go:1.1.1

ricxapp-hw-go-6bd8ff7d6f  nexus3.o-ran-sc.org:10002/o-ran-sc/ric-app-hw-go:1.1.1  replicas=1 ready=1

NAME                             READY   STATUS    RESTARTS   AGE   IP             NODE
ricxapp-hw-go-6bd8ff7d6f-67hq5   1/1     Running   0          121m   10.244.0.133   shyama7004

NAME    NAMESPACE  REVISION  STATUS     CHART       APP VERSION
hw-go   ricxapp    4         deployed   hw-go-1.0.0 1.0
```

</details>

<details>
<summary>Issues and fixes</summary>

**Symptom**  
Rollout stuck: `1 old replicas are pending termination...`

**Cause**  
Old pod was still pulling from `:10004` and wouldn’t exit.

**Fix**
```bash
kubectl -n ricxapp set image deploy/ricxapp-hw-go \
  hw-go=nexus3.o-ran-sc.org:10002/o-ran-sc/ric-app-hw-go:1.1.1

# optional pre-pull
sudo ctr -n k8s.io images pull nexus3.o-ran-sc.org:10002/o-ran-sc/ric-app-hw-go:1.1.1 || true

# recycle the bad pod
BAD=$(kubectl -n ricxapp get pods -o name | grep hw-go | head -n1)
[ -n "$BAD" ] && kubectl -n ricxapp delete "$BAD" --force --grace-period=0

kubectl -n ricxapp rollout status deploy/ricxapp-hw-go --timeout=3m
```
</details> 

### Step-12: Checked platform health via ingress

<details>
<summary>Commands Ran</summary>

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
NODE_IP=$(hostname -I | awk '{print $1}')
curl -s -o /dev/null -w "%{http_code}\n" http://$NODE_IP:32080/appmgr/ric/v1/health/ready
````

</details>

<details>
<summary>Output</summary>

<img width="551" height="81" alt="Screenshot 2025-08-22 at 11 11 49" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/12.png" />
</details>

<details>
<summary>Issues and fixes</summary>

**Symptom**  
Kong proxy service is `LoadBalancer` with `<pending>`.

**Cause**  
Bare-metal VM has no cloud load balancer.

**Fix**  
Use NodePort `32080` on the node IP:
```bash
NODE_IP=$(hostname -I | awk '{print $1}')
curl -s -o /dev/null -w "%{http_code}\n" http://$NODE_IP:32080/appmgr/ric/v1/health/ready
# expect: 200
```
</details>

### Step-13: Verified final xApp state

<details>
<summary>Commands Ran</summary>

```bash
kubectl -n ricxapp get deploy ricxapp-hw-go \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'

kubectl -n ricxapp get pods -o wide | grep hw-go
```

</details>

<details>
<summary>Output</summary>

<img width="723" height="41" alt="Screenshot 2025-08-22 at 11 12 33" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/13.png" />
</details>


### Step-14: Snapshot services & releases

<details>
<summary>Commands Ran</summary>

```bash
helm list -A
kubectl get pods -n ricplt
kubectl get pods -n ricinfra
kubectl get svc -A
```

</details>

<details>
<summary>Outputs</summary>

<img width="890" height="177" alt="Screenshot 2025-08-22 at 11 14 15" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/14.1.png" />
<img width="595" height="257" alt="Screenshot 2025-08-22 at 11 13 52" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/14.2.png" />
<img width="856" height="437" alt="Screenshot 2025-08-22 at 11 13 27" src="https://github.com/EdoItachi/ric-asrun/blob/main/screenshots/14.3.png" />
</details>


### E2TERM details for sims

<details>
<summary>Commands Ran</summary>

```bash
echo "E2TERM SCTP: $NODE_IP:32222"
kubectl -n ricplt get svc service-ricplt-e2term-sctp-alpha -o wide
```

</details>

> NodePort `32222` is available for sims (`$NODE_IP:32222`).

---

## Conclusion

I deployed Near-RT RIC (j-release) on a single-node VM, verified ingress health (`HTTP 200`), and successfully onboarded the `hw-go` xApp with `dms_cli`. The Deployment image was patched to `nexus3.o-ran-sc.org:10002/o-ran-sc/ric-app-hw-go:1.1.1`, and the final state shows one healthy Running pod. `helm list`, `kubectl get pods`, and `kubectl get svc` confirm that the platform is stable. E2TERM is reachable for sims at `$NODE_IP:32222`, and the system is ready for further xApp testing.

```
Created by shyama7004
