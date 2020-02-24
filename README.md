# GitDocs
A GitOps approach for hosting self-updating docs on Kubernetes - similar to Netlify.

With GitDocs you can create markdown documents in a central git repository and easily access them on a Kubernetes cluster. The documents will self-update as changes are detected in the git repo.

GitDocs is not an application, it is the "glue" between 3 microservices ([git-sync](https://github.com/kubernetes/git-sync), [hugo](https://gohugo.io), [nginx](https://www.nginx.com)) that gains superpowers when run in a pod on Kubernetes.

![](https://i.imgur.com/VIe32Ai.png)


For GitDocs to contain the entire documentation lifecycle in a pod, you need:
- access to a git repo
- access to a kubernetes cluster (v1.14 or later)

GitDocs runs anywhere, including on-prem, and supports private or public git repositories.

## Quick start

1) Clone the repository.
    ```
    git clone https://github.com/jimangel/GitDocs
    cd GitDocs
    ```
1) Apply the configmap and deployment.

    The `nginx.conf` file exists as a separate configmap to allow you to tweak any settings.

    ```
    kubectl apply -f examples/base/nginx-configmap.yaml
    kubectl apply -f examples/base/gitdocs-deployment.yaml
    ```

1) Use a kubectl proxy to preview.
    ```
    kubectl port-forward deployment/blog 80:8080
    ```

    Open a browser to http://localhost and browse.

## Going further

GitDocs's Quick Start purposefully excludes Ingress strategies; you can create your own ingress and TLS strategy. Also, by abstracting the nginx config, you can provide the SSL configuration to the pod.

## TODO:

Priority #1: Security
- Create network policies to isolate this pod
- Investigate any limitations on building images from non-root & scratch
- Add more hardening/vuln scanning

Priority #2: Makefile
- Create a Makefile for deploying, building, and updating

Other:
- Create a template for docs
- Add more examples (k8s.io, jimangel.io, etc)
- Improve all subsequent content
- Add additional kustomize examples for using your own images or setting up git-sync with an SSH key


## Apply with kustomize (-k) to make GitDocs your own

(coming soon)

Kustomize is a manifest generator built in to `kubectl` since version 1.14. If you're not familiar with Kustomize, note the following:

- The entire deployment lives in `example/base`. Please don't modify those files.
- Any CHANGES live in `example/overlays/hello-world`. This lets kustomize merge my changes with the core documents before deploying.
- You can have as many different `overlays` as you'd like while sharing the same `base`.

Taking all of the defaults, we can build an example site from a template:

```
kubectl apply -k examples/overlays/hello-world/
```

## Private repos

(coming soon)

## Troubleshooting

```
## general ##
# EVENTS: kubectl get events -n <NAMESPACE>

## git-sync ##
# LOGS: kubectl logs -f <POD NAME> -c git-sync
# EXEC: kubectl exec -it <POD NAME> -c git-sync /bin/sh
# DOCKER: docker run --rm -it \
    -v /tmp/git-data:/tmp/git --entrypoint /bin/sh \
    k8s.gcr.io/git-sync:v3.1.5

## hugo ##
# LOGS: kubectl logs -f <POD NAME> -c hugo
# EXEC: kubectl exec -it <POD NAME> -c hugo /bin/sh
# DOCKER: docker run --rm -it \
    -v /tmp/git-data:/tmp/git --entrypoint /bin/sh \
    klakegg/hugo:0.65.2-ext-alpine
    
## nginx ##
# LOGS: kubectl logs -f <POD NAME> -c nginx
# EXEC: kubectl exec -it <POD NAME> -c nginx /bin/sh
# CONFIG: kubectl edit cm nginx-conf -o yaml
```

## Changing Hugo versions

Change the hugo image tag in `examples/base/gitdocs-deployment.yaml`. For example: `klakegg/hugo:**0.65.2**-ext-alpine`. 

For more information, see: https://github.com/klakegg/docker-hugo

## Thanks

A lot of this is built on top of the example present in the git-sync repo: https://github.com/kubernetes/git-sync/tree/master/demo
