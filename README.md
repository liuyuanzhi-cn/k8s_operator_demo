

# task1
> 

```shell
# create a CustomResourceDefinition
kubectl apply -f pdf-crd.yaml
# create a PdfDocument
kubectl apply -f pdfdocument.yaml
# test
kubectl api-resources | grep pdf
kubectl get pdf

kubectl proxy --port=8080
curl localhost:8080/apis | grep k8s.startkubernetes.com
curl localhost:8080/apis/k8s.startkubernetes.com/v1/namespaces/default/pdfdocuments

```

# task2
> [youtube video](https://www.youtube.com/watch?v=q7b23612pSc)

```shell
mkdir pdfcontroller
cd pdfconcontroller
go mod init k8s.startkubernetes.com/v2
kubebuilder init
kubebuilder create api --group k8s.startkubernetes.com --version v2 --kind PdfDocument
make manifest
kubectl apply -f config/crd/bases/k8s.startkubernetes.com.my.domain_pdfdocuments.yaml

```

# task3

> 1. [youtuebe viedo: How to build a Kubernetes Webhook | Admission controllers](https://www.youtube.com/watch?v=1mNYSn2KMZk)
> 2. [sourse code](https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/kubernetes/admissioncontrollers/introduction)


```shell
mkdir tls
vim ca-config.json
vim ca-csr.json

docker run -it --rm -v ${PWD}:/work -w /work debian bash

cd /etc/apt/sources.list.d
cp debian.sources debian.sources.bak
sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" debian.sources
apt-get update && apt-get install curl -y && apt-get install openssl -y
# TODO: download in browse, network timeout
curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssl_1.5.0_linux_amd64 -o /usr/local/bin/cfssl
curl -L https://github.com/cloudflare/cfssl/releases/download/v1.5.0/cfssljson_1.5.0_linux_amd64 -o /usr/local/bin/cfssljson
chmod +x /usr/local/bin/cfssl
chmod +x /usr/local/bin/cfssljson

# ( container )generate certificate in /tmp
cfssl gencert \
  -ca=/tmp/ca.pem \
  -ca-key=/tmp/ca-key.pem \
  -config=./tls/ca-config.json \
  -hostname="example-webhook,example-webhook.default.svc.cluster.local,example-webhook.default.svc,localhost,127.0.0.1" \
  -profile=default \
  ./tls/ca-csr.json | cfssljson -bare /tmp/example-webhook
# ( container )make a secret, generate example-webhook-tls.yaml in ./tls
cat <<EOF > ./tls/example-webhook-tls.yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-webhook-tls
type: Opaque
data:
  tls.crt: $(cat /tmp/example-webhook.pem | base64 | tr -d '\n')
  tls.key: $(cat /tmp/example-webhook-key.pem | base64 | tr -d '\n') 
EOF
# ( container )generate webhook.yaml
#generate CA Bundle + inject into template
ca_pem_b64="$(openssl base64 -A <"/tmp/ca.pem")"

sed -e 's@${CA_PEM_B64}@'"$ca_pem_b64"'@g' <"webhook-template.yaml" \
    > webhook.yaml

```


# Reference

1. [controller-runtime project in github](https://github.com/kubernetes-sigs/controller-runtime)
2. [controller api document](https://pkg.go.dev/sigs.k8s.io/controller-runtime/pkg/manager#section-documentation)
