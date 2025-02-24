- install docker-registry with docker.local

```
#local-path storage class 설치

kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.31/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

#docker registry helm chart 설정

helm repo add twuni https://helm.twun.io

cat << EOF > values.yaml
replicaCount: 1

ingress:
  enabled: true
  className: traefik
  path: /
  hosts:
    - docker.local
  tls:
    - secretName: docker-registry-tls
      hosts:
        - docker.local
persistence:
  accessMode: 'ReadWriteOnce'
  enabled: true
  size: 10Gi
EOF

helm upgrade -i docker-registry docker-registry-2.2.3.tgz -f values.yaml -n registry --create-namespace

#인증서 / 인그레스 설정

openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
  -keyout docker.key -out docker.crt -subj '/CN=docker.local' \
  -addext 'subjectAltName=DNS:docker.local'

kubectl create secret tls docker-registry-tls --key docker.key --cert docker.crt -n registry

# 노드 마다 적용
scp docker.key node-01.local:/usr/local/share/ca-certificates
scp docker.crt node-01.local:/usr/local/share/ca-certificates

ssh node-01.local update-ca-trust # update-ca-certificates (Ubuntu)

/etc/containerd/config.toml

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          sandbox_mode = "podsandbox"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
      [plugins."io.containerd.grpc.v1.cri".registry]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
          [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
            endpoint = ["https://docker.local"]
            [plugins."io.containerd.grpc.v1.cri".registry.configs."docker.local".tls]
              ca_file = "/usr/local/share/ca-certificates/docker.crt"
              cert_file = "/usr/local/share/ca-certificates/docker.crt"
              key_file = "/usr/local/share/ca-certificates/docker.key"

```

- install gitea with gitea.local
- install tekton, tekton dashboard, argocd
- import opentelemetry java example
- build applications
- install lgtm-stack 

