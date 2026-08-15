## Hacking pods attached to the Host PID namespace

**[HostPID](https://kubesec.io/basics/spec-hostpid)** allows a pod to access all the processes running on the host and could allow an attacker to take malicious action. When paired with ```ptrace``` this can be used to escalate privileges outside of the container. When paired with a privileged container, the pod can see all of the processes on the host. An attacker can enter the ```init``` system (**[PID 1](https://denibertovic.com/posts/containers-and-signal-handling-why-you-need-to-care-about-pid-1/)**) on the host. From there, hackers can execute a shell and continue to escalate privileges to root. In this workflow, we will perform basic lateral credential harvesting via insecure process namespace sharing.
<br/><br/>
Download the script and make it executable:
```
wget https://raw.githubusercontent.com/ndouglas-edera/hostpid-risk/refs/heads/main/hostpid-risk.sh
chmod +x hostpid-risk.sh
```
Run the demo script:
```
./hostpid-risk.sh
```

<img width="1506" height="866" alt="Screenshot 2026-08-15 at 21 54 38" src="https://github.com/user-attachments/assets/64e7c073-4b2b-4eff-b821-9d8f15d978b9" />

<img width="1506" height="866" alt="Screenshot 2026-08-15 at 21 54 46" src="https://github.com/user-attachments/assets/ebdb7389-dfbf-4c1b-8928-241c8cc8348c" />

Create a ```privileged``` pod with ```hostPID``` set to ```true```:
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: inspector-pod
spec:
  hostPID: true
  containers:
  - name: inspector
    image: alpine:latest
    command: [\"/bin/sh\", \"-c\", \"while true; do sleep 3600; done\"]
    securityContext:
      privileged: true
EOF
```

Confirm the privileges of your running workloads:
```
kubectl get pods -A -o custom-columns=\
"NAMESPACE:.metadata.namespace",\
"POD:.metadata.name",\
"CONTAINER:.spec.containers[*].name",\
"HOST_PID:.spec.hostPID",\
"HOST_NET:.spec.hostNetwork",\
"PRIVILEGED:.spec.containers[*].securityContext.privileged"
```

<img width="1506" height="508" alt="Screenshot 2026-08-15 at 22 33 32" src="https://github.com/user-attachments/assets/654ab802-6551-4e29-b1f9-a47528555b93" />

