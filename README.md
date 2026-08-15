# hostpid-risk
Lateral credential harvesting via process namespace sharing
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
"NAME:.metadata.name",\
"HOST_PID:.spec.hostPID",\
"PRIVILEGED:.spec.containers[*].securityContext.privileged"
```

<img width="1506" height="428" alt="Screenshot 2026-08-15 at 22 17 08" src="https://github.com/user-attachments/assets/94ddcee7-4e51-4288-bbea-dd19b063172c" />
