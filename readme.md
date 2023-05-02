# prerequisites

Repository main goal is to get reused by author and, maybe, by people who find it useful. Currently author's setup is baremetal home server with [k3s](https://k3s.io) installed on it. So whole repository for sure works on k3s if folow installation guide below.<br>
As result of effort done you gonna get launched instance of application, with domain you configure and ssl above it

# installation
- Get the repo files, place them whenever it needed
- (optional) change namespace<br>
for handiness namespace in repo is "appnamespace", so it can be replaced in any comfortable way by mass replace. If you want use default one - just remove namespace from every manifest from metadata.<br>
Keep in mind if you change namespace you will have to tweak accordingly some commands below
- (optional) change name<br>
same as with namespace app name in repo "appname". same can be replaced by mass replace
- apply namespace by `kubectl create namespace appnamespace`<br>
heads up in here. if you changed namespace - tweak command accordingly
- (optional) do `docker login`
- `kubectl create secret generic dockerhub-secret --from-file=.dockerconfigjson={yourPath}/.docker/config.json --type=kubernetes.io/dockerconfigjson --namespace=appnamespace`
- `kubectl apply -f deployment.yaml`<br>
make sure it works (get pods) - if not, something of previos was done wrong. If error with image pull, cause no latest tag expected in your set up you may fix it by optional commands below
- (optional) `kubectl -n appnamespace set image deployment/appname nginx={repo}:{tag}
- (optional) `kubectl -n appnamespace set image deployment/appname php={repo}:{tag}
- `kubectl apply -f service.yaml`
- edit issuer.yaml by placing your mail in placeholder
- `kubectl apply -f issuer.yaml`
- `kubectl apply -f middleware.yaml`
- (optional) enabling https redirect `kubectl apply -f https-middleware.yaml`
- (optional) if applied https-middleware uncomment it in ingress.yaml before applying
- edit ingress.yaml by replacing placeholders of domain in there to preferred one
- `kubectl apply -f ingress.yaml`
- After everything done need to assure that certificate is fine by `kubectl -n appnamespace get cert` to see its READY in true status. If not wait ~5 min, recheck and if still not refer to troubleshooting url below
- If previous step is fine need to edit issuer.yaml by removing server and uncommenting #server. Reapply issuer and ingress.<br>
Personally me did remove secrets by `kubectl -n appnamespace delete secret/ingress-secret` and `kubectl -n appnamespace delete secret/issuer-secret`
- Enjoy your service with https configured
- (optional) If you use secrets do `cp secrets.yaml.dist secrets.yaml` and fill data by any secrets you want to use, [according to manual](https://kubernetes.io/docs/concepts/configuration/secret/). For those who read diagonally (as i do):
  - keep in mind that secrets have to be [base64-encoded strings](https://kubernetes.io/docs/concepts/configuration/secret/#restriction-names-data).
  -  To apply them [here's example how to](https://kubernetes.io/docs/concepts/configuration/secret/#use-cases)
- (optional) if you apply secrets you may apply (as i do) them as env variables in container. check `deployment.yaml`, uncomment and tweak according to own needs example in there

# knowledge sources
- https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
- https://cert-manager.io/docs/troubleshooting/acme/
- https://kubernetes.io/docs/concepts/configuration/secret/
