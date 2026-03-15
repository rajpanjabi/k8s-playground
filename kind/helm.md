## Helm

## Need for Helm

Before Helm, if we wanted to deploy our resources/objects like Deployments, Statefulsets or services, we had to individually create files and apply them in a given topological order, which works for a small scale project, but becomes error-prone and cumbersome for large scale. Also if we had to make even a small change, we had to find the appropriate file and modify and rebuild it. 

**Enters Helm**

A package manager for k8s.
We can create(install) as well as delete(uninstall) the entire application with just one command, no need to individually create objects.  It also simplifies tracking the entire app in one centralised place, editing values file to make change in the entire project and not editing each file separately, simplifying rollbacks and upgrades.

- Install helm ```bash brew install helm```

## Core Components


```helm create chart_name```

Creates this structure:
```
myapp/
├── Chart.yaml            # Chart metadata (name, version)
├── values.yaml           # Default config values
├── templates/            # K8s manifests with Go templating
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   ├── _helpers.tpl      # Reusable template snippets
│   └── NOTES.txt         # Post-install message
└── charts/               # Dependencies (sub-charts)
```

3 Basic Components (The "Big Three")

- Chart: A bundle of YAML templates. It’s the "source code" of your package. (Metadata like name, version, description, type)

- Values: A file (values.yaml) that contains the specific data you want to inject into the templates (e.g., image: my-app:v2.0).

- Release: A running instance of a Chart in your cluster. If you install the "Redis" chart twice, you have two separate Releases.

## To create/run

After creating files
```bash
helm install release_name directory
helm install first-release .

helm list # list all releases h

helm uninstall release_name # this deletes all the resources of that release

helm lint . # lint check or looks for syntax issues

helm template .  # shows the objects file that will be deployed with values incorporated from values.yaml

helm install release_name . --dry-run # this would not actually deploy but chekc any error that could happen in runtime i.e while deploying this release 

```

## Using already existing charts

Just like github repository, there exists a helm repository with various charts already defined. We can use it in following manner.

```bash
helm repo list # shows all repository installed in our cluster
# Artifact Hub has all helm repos
helm repo add repo_name repo_url # this adds the repo to our cluster
helm install release_name repo_name/release_name --version version_nos # to actually deploy that release  

# The above commands are used to directly use the existing charts, if we want to make changes, we need to pull those templates from the repos and update the files

helm pull --untar repo_name/release_name --version version_nos
helm pull --untar bitnami/wordpress --version 22.4.17

# We can update the values by creating a custom-values.yaml file and then upgrading the release
helm upgrade release_name .  --values custom-values.yaml

# To see history of releases
helm history RELEASE_NAME

# To rollback to a specific verison
helm rollback RELEASE_NAME REVISION

```

## Functions in helm

Whenever using {{- }}, add a hypen, so in compilation the line is disregarded and no empty space is left


To customize the files/manifest, we take input/config in the values.yaml file, but to make sure those adhere to the requirments/syntax, we can make use of functions inside {{}} when using those values in the manifests.


For instance, lets consider a case where we want the name of the deployment to be passed in upper-case. We can do {{ .Values.deployment.name | upper }}


Here we use a pipe operator after fetching the name from values file and passing it to upper function which converts the value to upper case. 


For multiple functions, we pipe them one after other as shown below:


{{ .Values.deployment.name | title | quote }} , here we first fetch values, then convert it to title case, followed by applying quotes on it.


Similarly, if we want a default value or a required value, we can do it in following manner:

- {{ default "default-value" .Values.deployment.name }} , here it would use defualt value if no value is configured in values.yaml for name field.

- {{ required "This value is required" .Values.deployment.name }} we declare it as required and add a message that is displayed when no value is configured

## If-else in Helm (Conditional Statements)

If we want to use different images based on the environment (pord, dev, test), we can use conditional statements to achieve it.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: webapp
    labels: 
        app: webapp
spec:
    replicas: 2
    selector: 
        matchLabels:
            app: webapp
    template:
        metadata:
            labels:
                app: webapp
        spec:
            containers:
                {{- if eq .Values.environment "prod" }}
                - image: webapp:prod
                {{- else if eq .Values.environment "dev" }}
                - image: webapp:dev
                {{- else }}
                - image: webapp:test
                {{- end }}
                  name: webapp
```

eq is used for equality and end is used finally to close the if-else statements

## with statements

In the manifest file, we are repeating .Values.field.subfield multiple times, which makes our code less readable and also redundant. To solve this, we use with statement to set the scope for the file, see an example below for better understanding:

```yaml
{{- with .Values.deployment }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .name }}
  namespace: {{ .name }}
spec:
  replicas: {{ .replicas }}                    
  selector:
    matchLabels:
      app: {{ .name }}
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
    #   to use value outside of deployment scope and fall back to root  use $.Values...
        - name: {{ $.Values.container.name }}
        # going more deeper in scope 
        {{- with .image}}
          image: {{ .repository }}:{{ .tag }}
        {{- end }}
        {{- with .ports }}
          ports:
            - containerPort: {{ .port1 }}
            - containerPort: {{ .port2 }}
        {{- end }}
          
{{- end }}
         
```

## Range (iteration like for loop)

1) 
    ```yaml
    <!-- values -->
    hosts:
    - api.example.com
    - api.backup.com
    ```

    ```yaml
    rules:
    {{- range .Values.hosts }}
    - host: {{ . }}    # "." refers to current item
    {{- end }}
    ```

2) 
    ```yaml
    <!-- values -->
    env:
    - name: NODE_ENV
        value: production
    - name: LOG_LEVEL
        value: info
    - name: PORT
        value: "8080"
    ```

    ```yaml
    env:
    {{- range .Values.env }}
    - name: {{ .name }}
        value: {{ .value }}
    {{- end }}

    ```