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

DONT USE DEFAULT NAMESPACE

oc port-forward -n elastic-demo service/elasticsearch-quickstart-es-http 9200
PASSWORD=$(oc get secret elasticsearch-quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
echo $PASSWORD
curl -u "elastic:$PASSWORD" -k "https://localhost:9200"