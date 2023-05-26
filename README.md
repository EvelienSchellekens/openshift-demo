# Elastic on OpenShift

## Install OC tools

Download oc zip file
Extract file
Move oc tool to a directory in your path (`echo $PATH`)
`sudo mv oc /usr/local/bin`
Allow it to execute: System Settings > Privacy Security > Allow oc

## Login to cluster

To allow for insecure connection, add this flag `--insecure-skip-tls-verify=true` to the command.
You can copy the command from the OpenShift UI
`oc login --token=<token> --server=https://<url>:6443 --insecure-skip-tls-verify=true`

## Install components

> WARNING: Don't use a default namespace

```shell
oc apply -f elasticsearch.yml

oc apply -f kibana.yml

oc create serviceaccount elastic-agent -n test
oc adm policy add-scc-to-user hostaccess -z elastic-agent -n test
oc adm policy add-scc-to-user hostmount-anyuid -z elastic-agent -n test
oc adm policy add-scc-to-user privileged -z elastic-agent -n test

oc apply -f fleet.yml

oc apply -f agent.yml

oc apply -f apm-server.yml

oc apply -f pet-clinic.yml

oc apply -f pet-clinic-service.yml
```

## Access Elastic components

```shell
oc port-forward -n elastic-demo service/elasticsearch-quickstart-es-http 9200

PASSWORD=$(oc get secret elasticsearch-quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')

echo $PASSWORD

curl -u "elastic:$PASSWORD" -k "https://localhost:9200"
```

Port-forward Kibana:

```shell
oc port-forward service/kibana-quickstart-kb-http 5601
```