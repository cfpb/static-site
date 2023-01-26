# Static Site Chart
This chart is used to simplify the deployment
of static sites that are hosted on a Git repository,
URL, or S3 bucket.


## How this works

The Pod has 2 containers, an init container, and the web container (nginx).
The init container will clone/download the specified repository and 
extract/copy the static files into an ephemeral directory. 
This directory is then mounted to the web container 
(nginx `/usr/share/nginx/html`).


## Usage
You will need to determine which method you will use to obtain
the static files.

Method is selected via `init.method`. Valid options are
`git`, `wget`, or `s3`. Default is `git`.

Refer to [values.yaml](./charts/static-site/values.yaml) for
the full set of overrides, including Ingress and Ambassador
Mapping.

### Git Value Overrides
The following values are used to drive the cloning of the
Git repository.

`init.git.repository` - The repository URL to clone

`init.git.ref` - The ref to checkout on clone (if any)

`init.git.subPath` - The sub path to root of the static files 
(if not the root of the repository) 

```bash
helm upgrade --install --wait <your-release> \
  --set init.git.repository=<your-repository-url> \
  --set init.git.ref=<your-ref> --set init.git.subPath=<static-subPath> \
  oci://ghcr.io/cfpb/static-site
```


### Wget Value Overrides
The following values are used to drive the downloading of
an archive from a URL, and extracting static root to `/static`.

`init.wget.url` - The URL to the artifact to download.

`init.wget.targetFile` - Filename to save the artifact as 
(to be used for extraction). This is also accessible for the extraction
command via environment variable `$TARGET_FILE`. Default is `static.tar`.
This file is downloaded to `/tmp/$TARGET_FILE`.

`init.wget.extractCommad` - The command needed to extract the artifact
and copy static root to `/static`.
Default is `tar xvf /tmp/$TARGET_FILE -C /static`.
Available extraction tools are `tar`, `unzip`, `gunzip`, and more.

**NOTE:** You will need to escape `$` when using `$TARGET_FILE` in 
the extract command to avoid local variable expansion. 

```bash
helm upgrade --install --wait <your-release> \
  --set init.method=wget --set init.wget.url=<url-to-wget> \
  --set init.wget.extractCommand="tar xvf /tmp/\$TARGET_FILE -C /static" \
  oci://ghcr.io/cfpb/static-site
```

You can even chain commands if you need to extract a subdirectory instead.

```bash
helm upgrade --install --wait <your-release> \
  --set init.method=wget --set init.wget.url=<url-to-wget> \
  --set init.wget.extractCommand="tar xvf /tmp/\$TARGET_FILE -C /tmp/static 
  && cp -Rfp /tmp/static/subdir/* /static" \
  oci://ghcr.io/cfpb/static-site
```


### AWS S3 Value Overrides
The following values are used to drive the downloading of
an object from an S3 bucket, and extracting static root to `/static`.

**NOTE:** To use S3, you will need to attach a Service Account that
has the appropriate permissions to access the S3 bucket. This is done
via `serviceAccount.name`.

`init.s3.bucket` - The bucket name containing the artifact object

`init.s3.object` - Object path to the static files artifact.

`init.s3.targetFile` - Filename to save the artifact as
(to be used for extraction). This is also accessible for the extraction
command via environment variable `$TARGET_FILE`. Default is `static.tar`.
This file is downloaded to `/tmp/$TARGET_FILE`.

`init.s3.extractCommand` - The command needed to extract the artifact
and copy static root to `/static`.
Default is `tar xvf /tmp/$TARGET_FILE -C /static`.
Available extraction tools are `tar`, `unzip`, `gunzip`, and more.

**NOTE:** You will need to escape `$` when using `$TARGET_FILE` in 
the extract command to avoid local variable expansion. 

```bash
helm upgrade --install --wait <your-release> \
  --set serviceAccount.name=<aws-s3-service-account> \
  --set init.method=s3 --set init.s3.bucket=<bucket-name> \
  --set init.s3.object=<path-to-object> \
  --set init.s3.extractCommand="tar xvf /tmp/\$TARGET_FILE -C /static" \
  oci://ghcr.io/cfpb/static-site
```

You can even chain commands if you need to extract a subdirectory instead.

```bash
helm upgrade --install --wait <your-release> \
  --set serviceAccount.name=<aws-s3-service-account> \
  --set init.method=s3 --set init.s3.bucket=<bucket-name> \
  --set init.s3.object=<path-to-object> \
  --set init.s3.extractCommand="tar xvf /tmp/\$TARGET_FILE -C /tmp/static 
  && cp -Rfp /tmp/static/subdir/* /static" \
  oci://ghcr.io/cfpb/static-site
```
