# GByte.tech Helm Charts

This Helm chart deploys a FastAPI application on Kubernetes with optional static file serving capabilities.

## Installation

To install the chart with the release name `my-fastapi`:

```bash
helm repo add gbyte https://gbytetech.github.io/helm/
helm install my-fastapi gbyte/fastapi
```

## Configuration

The following table lists the configurable parameters of the FastAPI chart and their default values.

| Parameter               | Description                               | Default                      |
|-------------------------|-------------------------------------------|------------------------------|
| `replicaCount`          | Number of replicas                        | `1`                          |
| `image.repository`      | FastAPI image repository                  | `gcr.io/my-project/my-image` |
| `image.tag`             | FastAPI image tag                         | `""`                         |
| `image.pullPolicy`      | Image pull policy                         | `IfNotPresent`               |
| `service.type`          | Kubernetes Service type                   | `ClusterIP`                  |
| `service.port`          | Service port                              | `80`                         |
| `service.containerPort` | Container port                            | `8000`                       |
| `ingress.enabled`       | Enable ingress controller resource        | `false`                      |
| `static.enabled`        | Enable static file serving                | `false`                      |
| `static.source`         | Source path for static files in the image | `/app/static`                |
| `static.root`           | Root path for serving static files        | `/var/www/public/static`     |
| `static.prefix`         | URL prefix for static files               | `/static/`                   |

## Using the Static File Serving Feature

The FastAPI chart includes an optional feature to serve static files using Nginx. This is useful for serving assets like
images, CSS, and JavaScript files alongside your FastAPI application.

To enable static file serving:

1. Set `static.enabled` to `true` in your values file:

   ```yaml
   static:
     enabled: true
   ```

2. Configure the static file paths:

   ```yaml
   static:
     enabled: true
     source: "/app/static"  # Path to static files in your FastAPI image
     root: "/var/www/public/static"  # Path where Nginx will serve files from
     prefix: "/static/"  # URL prefix for static files
   ```

3. Ensure your FastAPI image contains the static files at the specified `source` path.

4. If you're using ingress, make sure to configure it to route requests with the static prefix to the Nginx service:

   ```yaml
   ingress:
     enabled: true
     hosts:
       - host: your-domain.com
         paths:
           - path: /
             pathType: Prefix
           - path: /static
             pathType: Prefix
   ```

With this configuration, requests to `http://your-domain.com/static/*` will be served by Nginx, while other requests
will be handled by your FastAPI application.

## Customization

To customize the chart, create a `values.yaml` file and specify your values:

```yaml
replicaCount: 3
image:
  repository: your-repo/fastapi-app
  tag: v1.0.0
```

Then, install the chart with:

```bash
helm install my-fastapi gbyte/fastapi -f values.yaml
```

For more information on configuring the chart, please refer to the comments in the `values.yaml` file.
