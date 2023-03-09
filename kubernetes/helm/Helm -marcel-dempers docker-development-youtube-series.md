# Helm -marcel-dempers docker-development-youtube-series


https://github.com/si67ghb/docker-development-youtube-series
forked from
https://github.com/marcel-dempers/docker-development-youtube-series


https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm


## Clone github repo git@github.com:si67ghb/docker-development-youtube-series.git


git clone git@github.com:si67ghb/docker-development-youtube-series.git

cd docker-development-youtube-series/kubernetes/helm/

## Create Chart
https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm#create-our-first-chart
mkdir temp && cd temp
helm create example-app



https://helm.sh/docs/topics/charts/#the-chart-file-structure



## CleanUp template 
 Delete Unwanted files
controlplane $ ls -l example-app/templates/
total 32
-rw-r--r-- 1 root root 1763 Mar  8 11:08 NOTES.txt
-rw-r--r-- 1 root root 1822 Mar  8 11:08 _helpers.tpl
-rw-r--r-- 1 root root 1856 Mar  8 11:08 deployment.yaml			<----- delete
-rw-r--r-- 1 root root  928 Mar  8 11:08 hpa.yaml					<----- delete
-rw-r--r-- 1 root root 2087 Mar  8 11:08 ingress.yaml				<----- delete
-rw-r--r-- 1 root root  373 Mar  8 11:08 service.yaml				<----- delete
-rw-r--r-- 1 root root  328 Mar  8 11:08 serviceaccount.yaml		<----- delete
drwxr-xr-x 2 root root 4096 Mar  8 11:08 tests						<----- delete

TPL='./example-app/templates/'
ls -l ${TPL}
rm -rf ${TPL}*.yaml ${TPL}tests

controlplane $ ls -l ${TPL}
total 8
-rw-r--r-- 1 root root 1763 Mar  9 08:08 NOTES.txt
-rw-r--r-- 1 root root 1822 Mar  9 08:08 _helpers.tpl

## Add Kubernetes files to our new Chart


https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm#add-kubernetes-files-to-our-new-chart
K8S='/root/docker-development-youtube-series/kubernetes/'

cp ${K8S}/deployments/deployment.yaml example-app/templates/
cp ${K8S}/services/service.yaml example-app/templates/
cp ${K8S}/configmaps/configmap.yaml example-app/templates/
cp ${K8S}/secrets/secret.yaml example-app/templates/

```

controlplane $ ls -l ${TPL}
total 24
-rw-r--r-- 1 root root 1763 Mar  9 08:08 NOTES.txt
-rw-r--r-- 1 root root 1822 Mar  9 08:08 _helpers.tpl
-rw-r--r-- 1 root root  210 Mar  9 08:14 configmap.yaml
-rw-r--r-- 1 root root 1553 Mar  9 08:14 deployment.yaml
-rw-r--r-- 1 root root  241 Mar  9 08:14 secret.yaml
-rw-r--r-- 1 root root  251 Mar  9 08:14 service.yaml

```
## 
Rendering template
https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm#test-the-rendering-of-our-template
```
controlplane $ helm template --help

Render chart templates locally and display the output.

Any values that would normally be looked up or retrieved in-cluster will be
faked locally. Additionally, none of the server-side testing of chart validity
(e.g. whether an API is supported) is done.

Usage:
  helm template [NAME] [CHART] [flags]
     - template name
	 - directory name
	 
 -f, --values strings               specify values in a YAML file or a URL
	 
```	 
helm template example-app example-app	



## Install app using  Chart
https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm#install-our-app-using-our-chart

helm install example-app example-app
 ***  list releases ***
helm list

```
controlplane $ helm list                           
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
example-app     default         1               2023-03-09 08:15:59.587923525 +0000 UTC deployed        example-app-0.1.0       1.16.0     
```

## Value injections for our Chart
https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm#value-injections-for-our-chart

vi example-app/values.yaml  

 ***  values.yaml ***

deployment:
  image: "aimvector/python"
  tag: "1.0.4"

*** deployment.yaml ***

vi ${TPL}deployment.yaml
image: {{ .Values.deployment.image }}:{{ .Values.deployment.tag }}

### upgrade our release
helm template example-app example-app
helm upgrade example-app example-app --values ./example-app/values.yaml

### see revision increased

helm list

```
controlplane $ helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
example-app     default         2               2023-03-09 08:26:05.224259179 +0000 UTC deployed        example-app-0.1.0       1.16.0 
```

## Make our Chart more generic
https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm#make-our-chart-more-generic


 changing name
 
insert name into values.yaml file

vi example-app/values.yaml
```
deployment:
  image: "aimvector/python"
  tag: "1.0.4"

name: example-app
``` 

For the following objects, replace example-deploy and example-app to inject: "{{ .Values.name }}"

sed -i s'/example-app/{{ .Values.name }}/' ./example-app/templates/deployment.yaml
sed -i s'/example-app/{{ .Values.name }}/' ./example-app/templates/configmap.yaml
sed -i s'/example-app/{{ .Values.name }}/' ./example-app/templates/service.yaml
sed -i s'/example-service/{{ .Values.name }}/' ./example-app/templates/service.yaml
sed -i s'/example-deploy/{{ .Values.name }}/' ./example-app/templates/deployment.yaml
sed -i s'/example-config/{{ .Values.name }}/' ./example-app/templates/configmap.yaml


Rename values.yaml to example-app.values.yaml Create our second app values file example-app-02.values.yaml

mv  example-app/values.yaml example-app/example-app.values.yaml
cp  example-app/example-app.values.yaml example-app/example-app-02.values.yaml



``` 
Usage:
  helm template [NAME] [CHART] [flags]
     - template name
	 - directory name
	 
 -f, --values strings               specify values in a YAML file or a URL
	 
```	 
helm template example-app-02 example-app --values ./example-app/example-app-02.values.yaml

helm install example-app-02 example-app --values ./example-app/example-app-02.values.yaml

*** error ***
```
controlplane $ helm install example-app-02 example-app --values ./example-app/example-app-02.values.yaml
Error: rendered manifests contain a resource that already exists. Unable to continue with install: Secret "mysecret" in namespace "default" exists and cannot be imported into the current release: invalid ownership metadata; annotation validation error: key "meta.helm.sh/release-name" must equal "example-app-02": current value is "example-app"
```

sed -i s'/mysecret/{{ .Values.name }}/' ./example-app/templates/secret.yaml
helm install example-app-02 example-app --values ./example-app/example-app-02.values.yaml

## Uninstall a Release
```
controlplane $ helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
example-app     default         2               2023-03-09 09:18:39.465020508 +0000 UTC deployed        example-app-0.1.0       1.16.0     
example-app-02  default         1               2023-03-09 09:32:44.985339434 +0000 UTC deployed        example-app-0.1.0       1.16.0     
controlplane $ k get po
NAME                             READY   STATUS    RESTARTS   AGE
example-app-9784d77b9-h8dgk      1/1     Running   0          55s
example-app-9784d77b9-r7mlz      1/1     Running   0          55s
example-deploy-9784d77b9-gdbrw   1/1     Running   0          19m
example-deploy-9784d77b9-q5842   1/1     Running   0          19m

controlplane $ helm uninstall example-app-02
release "example-app-02" uninstalled
controlplane $ k get po
NAME                             READY   STATUS        RESTARTS   AGE
example-app-9784d77b9-h8dgk      1/1     Terminating   0          10m
example-app-9784d77b9-r7mlz      1/1     Terminating   0          10m
example-deploy-9784d77b9-gdbrw   1/1     Running       0          28m
example-deploy-9784d77b9-q5842   1/1     Running       0          28m
```




sed -i 's/name: example-app/name: example-app02/' ./example-app/example-app-02.values.yaml 
helm template example-app02 example-app --values ./example-app/example-app-02.values.yaml
helm install example-app02 example-app --values ./example-app/example-app-02.values.yaml

```
controlplane $ helm ls
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
example-app     default         2               2023-03-09 09:18:39.465020508 +0000 UTC deployed        example-app-0.1.0       1.16.0     
example-app02   default         1               2023-03-09 09:53:26.395225123 +0000 UTC deployed        example-app-0.1.0       1.16.0     
controlplane $ k get po
NAME                             READY   STATUS    RESTARTS   AGE
example-app02-f49475f45-4vk9r    1/1     Running   0          47s
example-app02-f49475f45-7vjrj    1/1     Running   0          47s
```


## Trigger deployment change when config changes
https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm#trigger-deployment-change-when-config-changes

# deployment.yaml

kind: Deployment
spec:
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}

Annotation changes because of configmap was changed - this triggers rollout of new pods

https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments

vi example-app/templates/deployment.yaml


# rollout the change

cp example-app/example-app-02.values.yaml example-app/example-app-02v2.values.yaml
helm upgrade example-app-02  example-app --values ./example-app/example-app-02v2.values.yaml



resulting annotation:
```
  template:
    metadata:
      annotations:
        checksum/config: 2ee623f0979ac20598c4b902bb018713d22c344b3b2b7fae1dd1115a828d1650
```

## Helm flow control
https://github.com/si67ghb/docker-development-youtube-series/tree/master/kubernetes/helm#ifelse-and-default-values
You can also set default values in case they are not supplied by the values.yaml file.
This may help you keep the values.yaml file small

Replace  to below

controlplane $ cat example-app/templates/deployment.yaml  | grep -A 6 resources
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "256Mi"
            cpu: "500m"

```
      {{- if .Values.deployment.resources }}
        resources:
          {{- if .Values.deployment.resources.requests }}
          requests:
            memory: {{ .Values.deployment.resources.requests.memory | default "50Mi" | quote }}
            cpu: {{ .Values.deployment.resources.requests.cpu | default "10m" | quote }}
          {{- else}}
          requests:
            memory: "50Mi"
            cpu: "10m"
          {{- end}}
          {{- if .Values.deployment.resources.limits }}
          limits:
            memory: {{ .Values.deployment.resources.limits.memory | default "1024Mi" | quote }}
            cpu: {{ .Values.deployment.resources.limits.cpu | default "1" | quote }}
          {{- else}}
          limits:
            memory: "1024Mi"
            cpu: "1"
          {{- end }}
        {{- else }}
        resources:
          requests:
            memory: "50Mi"
            cpu: "10m"
          limits:
            memory: "1024Mi"
            cpu: "1"
        {{- end}}


```
