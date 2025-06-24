
Get all default notebooks images

```bash
oc get imagestream -n redhat-ods-applications
```


get details of an imagestream (notebook) to understand the details python version and dependencies

```bash
oc get imagestream s2i-generic-data-science-notebook -o yaml -n redhat-ods-applications
```

create a Pipfile with below info

```toml

[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[dev-packages]



[packages]
# Datascience and useful extensions
boto3 = "~=1.26.69"
jupyter-bokeh = "~=3.0.5"
elyra-python-editor-extension = "~=3.14.2"
kafka-python = "~=2.0.2"
matplotlib = "~=3.6.3"
numpy = "~=1.24.1"
pandas = "~=1.5.3"
plotly = "~=5.13.0"
scikit-learn = "~=1.2.1"
scipy = "~=1.10.0"
jupyterlab-lsp = "~=3.10.2"
jupyterlab-widgets = "~=3.0.5"
jupyter-resource-usage = "~=0.6.0"
jupyterlab-s3-browser = "~=0.10.1"

# Parent image requirements to maintain cohesion
jupyterlab = "~=3.5.3"
jupyter-server = "~=2.1.0"
jupyter-server-proxy = "~=3.2.2"
jupyter-server-terminals = "~=0.4.4"
jupyterlab-git = "~=0.41.0"
nbdime = "~=3.1.1"
nbgitpuller = "~=1.1.1"
# ---
thamos = "~=1.29.1"
wheel = "~=0.38.4"


# Additional libraries
xgboost = "~=1.7.4"


[requires]
python_version = "3.9"

```

Install pipenv

```bash
pip install pipenv
```


Create lock file

```bash
pipenv lock
```

create dockerfile based on some existing notebook image (or from scratch - below is based on an existing image)

```dockerfile
FROM quay.io/modh/odh-generic-data-science-notebook:v2

# Install Python packages and Jupyterlab extensions from Pipfile.lock
COPY Pipfile.lock ./

RUN echo "Installing softwares and packages" && micropipenv install && rm -f ./Pipfile.lock

# Fix permissions to support pip in Openshift environments
RUN chmod -R g+w /opt/app-root/lib/python3.9/site-packages && \
    fix-permissions /opt/app-root -P
```

build image and push to the registry

```bash
podman build . -t custom-datascience-image:pipfile

# logging in to the registry
podman login

# tag and push
podman tag custom-datascience-image:pipfile quay.io/jishikaw/custom-datascience-image:pipfile
podman push quay.io/jishikaw/custom-datascience-image:pipfile
```


Adding custom images to RHODS

From here, use the RHODS console to import the created custom images.

Go to “Settings -> Notebook images” on the left side of the RHODS console to move to the import screen and follow the steps.

Once done enable the custom image by going to Notebook image setting,
