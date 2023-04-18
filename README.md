# Falco Helm Charts
3 main areas for Falco:
Events (introduced by Kernel, SysCalls interceptions, ..)
Powerful Rules Engine where the stream of events is asserted
And Alerts are triggered when a rule is violated

![image](https://user-images.githubusercontent.com/24285128/230738454-b059681b-39ff-440f-9ce2-e28b0ccd62f4.png)

# Falco Rules ( Complete list here https://github.com/falcosecurity/rules/blob/main/rules_inventory/rules_overview.md)
  Falco is all based on rules. Example rules could be: 

1.   Installation of packages, libraries inside any container
2.   Creation, deletion, rename and modification of files and folders inside the container after it starts running
3.   Execution of binaries like bash, ssh, docker binary, debian binaries, vpn client, mail binaries
4.   Unusual outbound traffic
5.   Any ssh connection to/from container
6.   Change of container files from host machine
7.   Detect an attempt to start a pod with a container image outside of a list of allowed images.
8.   Detect an attempt to start a pod with a privileged container
9.   Attempt to attach/exec to a pod
10.   Creation of new namespace etc.
11.  Write Below root

![image](https://user-images.githubusercontent.com/24285128/230739391-f6ff01f8-82e5-4ebe-8aa6-7935629e8643.png)


# 1.Deployment
```
   helm install falco falco/  -n falco --set falcosidekick.enabled=true --set falcosidekick.webui.enabled=true
```
# 2.Verification

  a. Falco has a webserver that captures K8S events
  ```
    kubectl exec -it "$container" -- curl -s localhost:8765/healthz; echo
  ```
  b. Check with source of events is configured
  ```
  $ kubectl logs -n falco -l "app.kubernetes.io/name=falco" -c falco-driver-loader --tail=-1 | grep "* Running falco-driver-loader with"
  ```
  ### Output for kernel module
  * Running falco-driver-loader with: driver=module, compile=yes, download=yes
  * Running falco-driver-loader with: driver=module, compile=yes, download=yes

  ### Output for eBPF
  * Running falco-driver-loader with: driver=bpf, compile=yes, download=yes
  * Running falco-driver-loader with: driver=bpf, compile=yes, download=yes

# 3.Confirm the driver is properly installed
```
  $ kubectl logs -n falco -l "app.kubernetes.io/name=falco" -c falco-driver-loader --tail=-1 | grep -A 5 "* Success"
```
# 4.Trigger one of the Falco rules
```
  $ export POD_NAME=$(kubectl get pods --namespace falco -l "app.kubernetes.io/name=falco" -o jsonpath="{.items[0].metadata.name}")
  $ kubectl -n falco exec ${POD_NAME} -c falco  -- find /root -name "id_rsa"
```
  ### Check that Falco correctly intercepted the potentially dangerous command:
```
  $ kubectl logs -n falco -c falco -l "app.kubernetes.io/name=falco" | grep Warning
```
# 5. Event-generator
  If you’d like to check if Falco is working properly, we have the event-generator tool that can perform an activity for both our syscall and k8s audit   related rules.
  ```
  helm install event-generator event-generator -f event-generator/values.yaml --namespace event-generator
  ```
  
 # 6.Falco UI
 We have falcosidekick ui service which is clusterIP. You can change it to Nodeport or LoadBalancer to access it from outside the cluster
  ```
  kubectl -n falco port-forward service/falco-falcosidekick-ui 2802:2802
  ```
  ![image](https://user-images.githubusercontent.com/24285128/230738999-6da7ca77-a24b-4369-8b89-25fc5d0f933d.png)

 
