## ConfigMap & Secrets
ConfigMap and Secrets are resources used to manage configuration data and sensitive information, respectively. Here’s a concise breakdown of each:

ConfigMap
*  Purpose: Stores non-sensitive configuration data such as application settings, environment variables, and configuration files.
* Usage: Allows you to decouple configuration artifacts from container images, making your applications more portable and easier to manage.

Secrets
* Purpose: Stores sensitive data such as passwords, OAuth tokens, and SSH keys.
* Usage: Ensures that sensitive data is stored securely and can be accessed by Pods without hardcoding the information into images or configuration files.
* Data in Secrets is base64 encoded for storage but not encrypted by default.

### Task 1: Directly inject variables - Traditional Method
#### Imperative Method
```
kubectl run pod-imperative --image nginx --env "db_user=admin"
```
Exec into the pod and check the environment variable
```
kubectl exec -it pod-imperative -- bash
```
```
echo $db_user
```
![image](https://github.com/user-attachments/assets/90ede114-e875-4b49-9d0b-266fc95a8e88)

```
exit
```

#### Declartive Method
Create a YAML file to pass the environment variables
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: ws
  name: env-pod
spec:
  containers:
  - image: nginx
    name: ng-ctr
    ports:
    - containerPort: 80
    env:
    - name: db_user   #key
      value: admin
    - name: db_pwd
      value: "1234"
```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod env-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it env-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
env | grep db_
```
![image](https://github.com/user-attachments/assets/2213796f-194e-4849-bbce-1cec5fe1572f)
```
exit
```


### Task 2: Inject `ALL` variables from ConfigMaps(FromLiteral) into POD.
Create a ConfigMap
```
kubectl create cm cm-1 --from-literal=db_user=admin --from-literal=db_pwd=1234
```
```
kubectl get cm cm-1 -o yaml
```
![image](https://github.com/user-attachments/assets/b3bb49d0-f52f-45bc-85fc-a10f13e17a05)

Inject the ConfigMap into the Pod Yaml File
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: cm-1
```
```
kubectl apply -f env.yaml
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
env | grep db_
```
![image](https://github.com/user-attachments/assets/3cc392c1-cc56-489c-8df4-f14f4bfa36ae)
```
exit
```


### Task 3: Inject `PARTICULAR` variables from ConfigMaps(FromLiteral) into POD.
Create a ConfigMap
```
kubectl create cm cm-1 --from-literal=db_user=admin --from-literal=db_pwd=1234
```
```
kubectl get cm cm-1 -o yaml
```
Inject particular variable from the ConfigMap into the Pod Yaml File
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    env:
    - name: db_password
      valueFrom:
        configMapKeyRef:
          name: cm-1
          key: db_pwd
```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod web-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
env | grep db_
```
![image](https://github.com/user-attachments/assets/3ffc264e-9703-4b9c-befb-edb0bead4e1c)
```
exit
```


### Task 4: Inject variables from ConfigMaps(FromFile) into POD.
Create a file
```
vi token
```
```
This is CKAD Training. We are practicing Injecting variables from ConfigMaps(FromFile) into POD.
```
Create a ConfigMap. `--from-file=<filen-name>. This file name acts as the key`
```
kubectl create cm cm-1 --from-file=token         
```
```
kubectl get cm cm-1 -o yaml
```
Inject particular variable from the ConfigMap into the Pod Yaml File
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  containers:
  - image: httpd
    name: ctr-1
    ports:
    - containerPort: 80
    envFrom:
    - configMapRef:
        name: cm-1
```
```
kubectl apply -f env.yaml
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
env | grep token
```

### Task 5 : Injecting ConfigMap as volume mount

Create a file
```
vi token
```
```
This is CKAD Training. We are practicing Injecting variables from ConfigMaps(FromFile) into POD.
```
Create a ConfigMap
```
kubectl create cm cm-1 --from-file=token        
```
```
kubectl get cm
```
```
kubectl describe cm cm-1
```
Inject as volume mount
```
vi env.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: web
  name: web-pod
spec:
  volumes:
  - name: cm-volume
    configMap:
      name: cm-1
  containers:
  - image: httpd
    name: ctr-1
    volumeMounts:
    - name: cm-volume
      mountPath: /app
    ports:
    - containerPort: 80

```
```
kubectl apply -f env.yaml
```
```
kubectl describe pod web-pod
```
Enter the pod and check if the variable has been passed correctly or not
```
kubectl exec -it web-pod -- sh
```
```
cd /app
```
```
cat token
```

### Task 6 : Secret
Imperative
```
kubectl create secret generic secret-1 --from-literal=db_user=admin --from-literal=db_pwd=123
```
```
kubectl get secret
```
```
kubectl describe secret secret-1
```
Declrative
```
vi secret.yaml
```
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-credentials
type: Opaque
data:
  ##all below values are base64 encoded

  ## To encode and print
  ## echo -n '<value-to-be-encoded>' | base64

  ## To decode and print
  ## echo '<value-to-be-decoded>' | base64 -d

  ##rootpw is root
  rootpw: cm9vdAo=

  ##user is user
  user: dXNlcgo=

  ##password is mypwd
  password: bXlwd2QK
```
```
kubectl apply -f secret.yaml
```
```
kubectl get secrets
```
```
kubectl describe secrets mysql-credentials
```
You can inject the secret in all the three ways as above.

Injecting all values
```
vi sc-pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sc-pod
  name: sc-pod
spec:
  containers:
  - image: nginx
    name: sc-ctr
    envFrom:
    - secretRef:
        name: secret-1
    - secretRef:
        name: mysql-credentials
```
```
kubectl apply -f sc-pod.yaml
```
```
kubectl get po
```
```
kubectl exec -it sc-pod -- sh
```
```
echo $db_user
```
```
echo $db_pwd
```
```
echo $rootpw
```
```
echo $user
```
```
echo $password
```
```
env 
```