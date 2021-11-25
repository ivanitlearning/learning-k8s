# Creating a user in k8s cluster

This is how to create a user sarah and let her read, watch, list and update pods only

First generate a private key

```text
root@master:~/users# openssl genrsa -out sarah.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
.............................................+++++
.............................................+++++

root@master:~/users# cat sarah.key
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEAyOLifYNUGHVGF8Owbx7RFn0RHzUk2yVWQNJFa+mC/8hnq4t9
v2vrBDnH/yAJo8FyQLbLZqF6rF6LbCJGBMzEGusuYPVJQBq2cM4l821EtVOJW2J/
E6rFnodrsl2QZaaqh6crTj0kFtnJiidt7WvuCrldEsOco1lLRDoXpnwJwRIjGeWW
bnk2b9JqyION6yq6L9pG0IbQ8Xhamm8zG/dnZha9bspFZmFBN7BfD+auf9Ni/q/R
IWnNMMLu1SpCcuzUZ/1Cw9o+JOIAF4UqNjmy+ojQnu3hsMv2X9i8JUy6rlZt5L9y
/wWc5wBE/oClse6Y9smtf1GFynrnmJDfty7vdQIDAQABAoIBAQC1T8pavIDXfmmg
I1iIQsk2wfUFNMMqJF3gLajIyD8bO7cOJt19Oxsoejpfs3vf2EaV0CZvYEnHb6Hp
HFoUFPjezuvSSTEu8a0+zWGFf1bnThmIaLMHfjfVaKa0mywsAyyhOSml+RssoK79
ya62/pYgCcPOns0uxfZgAwq7kfJ/1ZQ4TWvGOt1Bpa+uTiCf8y2O+b/BWdjF9Wnw
8rrswqAarZK82PKZQytT4Vx8iYoYc8HXQPJPqW6dIdBiSWLyIbDf/Xq+Qyr0YNbf
ugaZCU57nB0Xe6xkpInvNuQbNVRLBqFGbVmar6ucuXDsa1KIC7NEISr9XtV1xtyY
R4g6ZPHhAoGBAOvr5WCp+njikWTQAP/8BhtbMWFuwpTOEjDPPCNlqwjHBsn4B2uU
HrvCopw23aGxz2Qy7Wvl2AeY5IFT83GLtOY/Hx4qWGe18MAUR/VCbfb0M73XNe8j
yufDlN3sxZmEJaR7vCz6UcTBrjN1iO+7hahgnJNH55FF+dUMV36LbmKdAoGBANn7
qhw+OkipVW2QRNNnxpBtQHKOyp431+I+RFWoWNswrjHe2UEnTMWF5R29FM9+453x
ERvkyHQpRqioe3JsOauLvvLVi1YhVdM+3pJJpiCF/ov8I6BWJfD6cghOEOJ4HoNW
OhqG7q9sPMfyv860Hk2+3Gdb66iubnpognv/+py5AoGBAIcC46zS+a0uc+hOhRP5
pYEIShUpLp+74nseTZswNpX6WC9DCvQMux3Wf/qIB4PeXwJHhsmlqmCGpdZBNeM4
AVl2rBc2QotveoxhzuBTmNyn2eh9fbcSM684pTvvoRF+p5Ae44yV4C+Ka2e1jp0r
Io0+ZLyAfMwNULEUtAmOP6idAoGBAKccOgNA6Wm+91DxYvI3ApDCUMACG+9DnGtD
lRud3dDb9w8gaql6OW7MASPVStjvzAvPPXCG6e2znwm5cDn+IhATKCX9873p/GPg
NL0tXQBd+RDUEXPf12JwfW9EeclEkQ/a0Nx5SQ6PCeG3hbgveXPcuBc87uL4JpYM
/MuXKEdxAoGBAOnFD5A6WyMxIHw4VjNyifT1+Xd8QPsmjU/BmTGNBpTaAjc1kaTP
vWaS1Sf8gKME8d5X4j5hGtAMdDrRB2sq/DyqapdflODDMNbqOTncCiBFq8vINx9R
yxGv6dJhRdI86Lll47pU+Ao/0noX/dxJ1vXrj00EP8B3GJMKTWLOiCIQ
-----END RSA PRIVATE KEY-----
```

Now create a CSR file

```text
root@master:~/users# openssl req -new -key sarah.key -subj "/CN=sarah" -out sarah.csr
root@master:~/users# openssl req -noout -text -in sarah.csr
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: CN = sarah
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:c8:e2:e2:7d:83:54:18:75:46:17:c3:b0:6f:1e:
                    d1:16:7d:11:1f:35:24:db:25:56:40:d2:45:6b:e9:
                    82:ff:c8:67:ab:8b:7d:bf:6b:eb:04:39:c7:ff:20:
                    09:a3:c1:72:40:b6:cb:66:a1:7a:ac:5e:8b:6c:22:
                    46:04:cc:c4:1a:eb:2e:60:f5:49:40:1a:b6:70:ce:
                    25:f3:6d:44:b5:53:89:5b:62:7f:13:aa:c5:9e:87:
                    6b:b2:5d:90:65:a6:aa:87:a7:2b:4e:3d:24:16:d9:
                    c9:8a:27:6d:ed:6b:ee:0a:b9:5d:12:c3:9c:a3:59:
                    4b:44:3a:17:a6:7c:09:c1:12:23:19:e5:96:6e:79:
                    36:6f:d2:6a:c8:83:8d:eb:2a:ba:2f:da:46:d0:86:
                    d0:f1:78:5a:9a:6f:33:1b:f7:67:66:16:bd:6e:ca:
                    45:66:61:41:37:b0:5f:0f:e6:ae:7f:d3:62:fe:af:
                    d1:21:69:cd:30:c2:ee:d5:2a:42:72:ec:d4:67:fd:
                    42:c3:da:3e:24:e2:00:17:85:2a:36:39:b2:fa:88:
                    d0:9e:ed:e1:b0:cb:f6:5f:d8:bc:25:4c:ba:ae:56:
                    6d:e4:bf:72:ff:05:9c:e7:00:44:fe:80:a5:b1:ee:
                    98:f6:c9:ad:7f:51:85:ca:7a:e7:98:90:df:b7:2e:
                    ef:75
                Exponent: 65537 (0x10001)
        Attributes:
            a0:00
    Signature Algorithm: sha256WithRSAEncryption
         3b:a3:20:a1:5c:56:5e:8a:b3:fc:10:f1:43:46:2d:e5:a9:ae:
         0d:1f:12:14:06:a4:d0:df:c1:cb:ef:93:6b:57:30:7f:f3:60:
         d9:89:8c:6c:1f:d6:68:71:7b:28:32:11:d4:df:cd:63:b1:53:
         94:ba:7d:32:85:8b:32:5b:e0:f3:b4:6b:8f:1b:e6:39:d6:7e:
         de:48:34:f1:6f:38:f0:23:a0:af:17:44:f7:9c:f5:b6:37:36:
         11:c1:c4:72:73:86:60:1c:be:85:02:6e:68:fe:ea:41:45:2f:
         5b:07:4c:4e:c9:ba:51:1e:bb:86:4a:1b:33:e5:d5:d9:0d:9e:
         1e:00:3a:08:89:b7:b4:48:d1:65:64:11:12:b6:8a:09:b2:ac:
         43:e6:89:02:c4:f3:0d:32:70:75:da:c7:3e:69:8c:51:99:3d:
         cc:15:43:18:d7:fe:a6:7a:c6:81:bb:2c:25:88:4e:04:46:65:
         2e:4d:4b:72:01:83:06:e8:f9:e7:f5:23:4d:61:62:7f:08:c0:
         9b:84:c5:3b:69:e3:ff:ee:b8:0e:a4:6a:79:47:da:4f:b5:54:
         48:6b:ab:d2:9b:1b:a7:29:0b:2f:cc:0b:85:42:e8:73:18:a3:
         39:36:e5:85:28:2a:b0:48:82:08:50:d9:f8:a6:19:e9:50:53:
         16:ce:92:40
```

Then we need this file in base64

```text
root@master:~/users# cat sarah.csr | base64 -w0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZUQ0NBVDBDQVFBd0VERU9NQXdHQTFVRUF3d0ZjMkZ5WVdnd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQQpBNElCRHdBd2dnRUtBb0lCQVFESTR1SjlnMVFZZFVZWHc3QnZIdEVXZlJFZk5TVGJKVlpBMGtWcjZZTC95R2VyCmkzMi9hK3NFT2NmL0lBbWp3WEpBdHN0bW9YcXNYb3RzSWtZRXpNUWE2eTVnOVVsQUdyWnd6aVh6YlVTMVU0bGIKWW44VHFzV2VoMnV5WFpCbHBxcUhweXRPUFNRVzJjbUtKMjN0YSs0S3VWMFN3NXlqV1V0RU9oZW1mQW5CRWlNWgo1Wlp1ZVRadjBtcklnNDNyS3JvdjJrYlFodER4ZUZxYWJ6TWI5MmRtRnIxdXlrVm1ZVUUzc0Y4UDVxNS8wMkwrCnI5RWhhYzB3d3U3VktrSnk3TlJuL1VMRDJqNGs0Z0FYaFNvMk9iTDZpTkNlN2VHd3kvWmYyTHdsVExxdVZtM2sKdjNML0Jaem5BRVQrZ0tXeDdwajJ5YTEvVVlYS2V1ZVlrTiszTHU5MUFnTUJBQUdnQURBTkJna3Foa2lHOXcwQgpBUXNGQUFPQ0FRRUFPNk1nb1Z4V1hvcXovQkR4UTBZdDVhbXVEUjhTRkFhazBOL0J5KytUYTFjd2YvTmcyWW1NCmJCL1dhSEY3S0RJUjFOL05ZN0ZUbExwOU1vV0xNbHZnODdScmp4dm1PZForM2tnMDhXODQ4Q09ncnhkRTk1ejEKdGpjMkVjSEVjbk9HWUJ5K2hRSnVhUDdxUVVVdld3ZE1Uc202VVI2N2hrb2JNK1hWMlEyZUhnQTZDSW0zdEVqUgpaV1FSRXJhS0NiS3NRK2FKQXNUekRUSndkZHJIUG1tTVVaazl6QlZER05mK3BuckdnYnNzSlloT0JFWmxMazFMCmNnR0RCdWo1NS9ValRXRmlmd2pBbTRURk8ybmovKzY0RHFScWVVZmFUN1ZVU0d1cjBwc2JweWtMTDh3TGhVTG8KY3hpak9UYmxoU2dxc0VpQ0NGRForS1laNlZCVEZzNlNRQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
```

Let's create a CSR object using this base64 input

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: sarah
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZUQ0NBVDBDQVFBd0VERU9NQXdHQTFVRUF3d0ZjMkZ5WVdnd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQQpBNElCRHdBd2dnRUtBb0lCQVFESTR1SjlnMVFZZFVZWHc3QnZIdEVXZlJFZk5TVGJKVlpBMGtWcjZZTC95R2VyCmkzMi9hK3NFT2NmL0lBbWp3WEpBdHN0bW9YcXNYb3RzSWtZRXpNUWE2eTVnOVVsQUdyWnd6aVh6YlVTMVU0bGIKWW44VHFzV2VoMnV5WFpCbHBxcUhweXRPUFNRVzJjbUtKMjN0YSs0S3VWMFN3NXlqV1V0RU9oZW1mQW5CRWlNWgo1Wlp1ZVRadjBtcklnNDNyS3JvdjJrYlFodER4ZUZxYWJ6TWI5MmRtRnIxdXlrVm1ZVUUzc0Y4UDVxNS8wMkwrCnI5RWhhYzB3d3U3VktrSnk3TlJuL1VMRDJqNGs0Z0FYaFNvMk9iTDZpTkNlN2VHd3kvWmYyTHdsVExxdVZtM2sKdjNML0Jaem5BRVQrZ0tXeDdwajJ5YTEvVVlYS2V1ZVlrTiszTHU5MUFnTUJBQUdnQURBTkJna3Foa2lHOXcwQgpBUXNGQUFPQ0FRRUFPNk1nb1Z4V1hvcXovQkR4UTBZdDVhbXVEUjhTRkFhazBOL0J5KytUYTFjd2YvTmcyWW1NCmJCL1dhSEY3S0RJUjFOL05ZN0ZUbExwOU1vV0xNbHZnODdScmp4dm1PZForM2tnMDhXODQ4Q09ncnhkRTk1ejEKdGpjMkVjSEVjbk9HWUJ5K2hRSnVhUDdxUVVVdld3ZE1Uc202VVI2N2hrb2JNK1hWMlEyZUhnQTZDSW0zdEVqUgpaV1FSRXJhS0NiS3NRK2FKQXNUekRUSndkZHJIUG1tTVVaazl6QlZER05mK3BuckdnYnNzSlloT0JFWmxMazFMCmNnR0RCdWo1NS9ValRXRmlmd2pBbTRURk8ybmovKzY0RHFScWVVZmFUN1ZVU0d1cjBwc2JweWtMTDh3TGhVTG8KY3hpak9UYmxoU2dxc0VpQ0NGRForS1laNlZCVEZzNlNRQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```

Then approve it

```text
root@master:~/users# k apply -f sarah.yaml
certificatesigningrequest.certificates.k8s.io/sarah created
root@master:~/users# k get csr
NAME    AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
sarah   2s    kubernetes.io/kube-apiserver-client   kubernetes-admin   24h                 Pending
root@master:~/users# k certificate approve sarah
certificatesigningrequest.certificates.k8s.io/sarah approved
root@master:~/users# k get csr
NAME    AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
sarah   62s   kubernetes.io/kube-apiserver-client   kubernetes-admin   24h                 Approved,Issued
```

Now we need to create a public cert for sarah

```text
root@master:~/users# openssl x509 -req -in sarah.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out sarah.crt -days 500
Signature ok
subject=CN = sarah
Getting CA Private Key
```

Now let's add this user to kubeconfig. Let's first check there isn't a user named sarah there.

```text
root@master:~/users# k config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.16.1.19:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

Let's create a role for sarah to allow her to get, list, watch, update pods

```text
root@master:~/users# k create role sarah-pod --verb=get,list,update,watch --resource=pods
role.rbac.authorization.k8s.io/sarah-pod created
root@master:~/users# k describe role sarah-pod
Name:         sarah-pod
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get list update watch]
```

Now bind this role to sarah

```text
root@master:~/users# k create rolebinding sarah-pod --role=sarah-pod --user=sarah
rolebinding.rbac.authorization.k8s.io/sarah-pod created
root@master:~/users# k describe rolebinding sarah-pod
Name:         sarah-pod
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  sarah-pod
Subjects:
  Kind  Name   Namespace
  ----  ----   ---------
  User  sarah
```

Note that even before creating user entries in kubeconfig we can test perms with `auth can-i`

```text
root@master:~/users# k auth can-i get pods --as sarah
yes
root@master:~/users# k auth can-i create pods --as sarah
no
```

Create the user entry in kubeconfig

```text
root@master:~/users# k config set-credentials sarah --client-certificate=sarah.crt --client-key=sarah.key
User "sarah" set.
root@master:~/users# k config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.16.1.19:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: sarah
  user:
    client-certificate: /root/users/sarah.crt
    client-key: /root/users/sarah.key
```

Then finally create the context for sarah

```text
root@master:~/users# k config set-context sarah-pod-adm --user=sarah --cluster=kubernetes
Context "sarah-pod-adm" created.
root@master:~/users# k config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://10.16.1.19:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
- context:
    cluster: kubernetes
    user: sarah
  name: sarah-pod-adm
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
- name: sarah
  user:
    client-certificate: /root/users/sarah.crt
    client-key: /root/users/sarah.key
```

Now we switch to context sarah-pod-adm but find we can't create pods but we can list them.

```text
root@master:~/users# k config use-context sarah-pod-adm
Switched to context "sarah-pod-adm".
root@master:~/users# k config get-contexts
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
          kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
*         sarah-pod-adm                 kubernetes   sarah
root@master:~/users# k get pods
No resources found in default namespace.
root@master:~/users# k run nginx --image=nginx
Error from server (Forbidden): pods is forbidden: User "sarah" cannot create resource "pods" in API group "" in the namespace "default"
```

