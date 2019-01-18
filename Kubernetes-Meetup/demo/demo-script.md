# Part 1
## Install books app example
curl https://run.linkerd.io/booksapp.yml | kubectl apply -f -

## port-forward webapp
kubectl port-forward svc/webapp 7000

## Click around see if you can detect the error
open http://localhost:7000

## Open up kubernetes dashboard
Note: Ensure you have a kubectl proxy running on port 8001

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

## Let's check out some of the booksapp logs to see if there is anything weird

stern -h

stern authors.* | less

# Part 2
## Install Linkerd
I already downloaded it, but its available at run.linkerd.io


## make sure everything is in good shape with Linkerd check
linkerd check --pre
- we run `check` to make sure we can:
- - Reach the Kubernetes API server, since we rely on that for service discovery etc
- - Check RBAC permissions to make sure we actually can install Linkerd, also to make sure we don't override an existing Linkerd install
- - Make sure we are using the right Linkerd version

## Now we can install Linkerd since everything passes.
- Linkerd install outputs a K8S yaml that install the various Linkerd control plane components
linkerd install | kubectl apply -f -

## Let's do a check to see if everything is righ with the world after the install
- When running Linkerd check without the `--pre` flag, we are checking that:
- - All the previous checks and making sure the newly install control plane can reach the K8S API
- - The control plane can reach Prometheus

## Let's open up the dashboard and see what Linkerd shows us

linkerd dashboard

As you can see we are able to see our booksapp deployment however, we don't see any metrics for our services becuase we haven't injected sidecar proxies into the booksapp deployments. So let's do that

## Inject Linkerd into booksapp
linkerd inject booksapp.yml | kubectl apply -f -

# So, everything worked great! Let's see if can get stats about our booksapp services
watch linkerd stats deploy

# Part 3
linkerd profile --open-api swagger/authors.swagger authors | kubectl apply -f -
