# Unlocking the Power of Elastic on OpenShift

> This is a DEMO, don't use this code in production please. :)

> Documentation on running ECK on OpenShift is located [here](https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-openshift.html).

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

## DEMO script

### Install operator

Click click click in the UI.

Check if it worked:

```shell
oc get pods -n openshift-operators
```

### Deploying Elasticsearch nodes

Create a new project first:

```shell
oc new-project demo --display-name 'OpenShift Demo' 
```

Apply manifest:

```shell
oc apply -f 1-elasticsearch.yml
```

Apply 30-day free trial:

```shell
oc apply -f 1-trial.yml
```

Check if it worked:

```shell
oc get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

oc port-forward -n demo service/elasticsearch-quickstart-es-http 9200

PASSWORD=$(oc get secret elasticsearch-quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')

echo $PASSWORD

curl -u "elastic:$PASSWORD" -k "https://localhost:9200"
```

### Deploy Kibana

Apply manifest:

```shell
oc apply -f 2-kibana.yml
```

Check if it worked:

```shell
oc get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName

oc port-forward service/kibana-quickstart-kb-http 5601

PASSWORD=$(oc get secret elasticsearch-quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')

echo $PASSWORD
```

Open `https://localhost:5601` in browser.

### Deploy Fleet Server

Create a new service account and apply privileges:

```shell
oc create serviceaccount elastic-agent -n demo

oc adm policy add-scc-to-user hostaccess -z elastic-agent -n demo
oc adm policy add-scc-to-user hostmount-anyuid -z elastic-agent -n demo
oc adm policy add-scc-to-user privileged -z elastic-agent -n demo
```

Apply manifest:

```shell
oc apply -f 3-fleet.yml
```

Check if it worked:

```shell
oc get pods
```

Go to Kibana and check enrolled agent.

### Deploy Elastic Agent as a DaemonSet

Apply manifest:

```shell
oc apply -f 4-agent.yml
```

Might need this hack:

```shell
oc get pods -n openshift-operators
oc delete po pod-name -n openshift-operators
```

Check if it worked:

```shell
oc get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName
```

Go to Kibana and check enrolled agents.

### Install APM Server

Apply manifest:

```shell
oc apply -f 5-apm-server.yml
```

### Install demo application 

Apply manifest:

```shell
oc apply -f 6-pet-clinic.yml
```

Check if it worked:

```shell
oc get pods

oc port-forward service/petclinic 8080
```

In browser go to `http://localhost:8080`

Go to Kibana and check APM service.