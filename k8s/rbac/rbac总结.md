rbac总结
https://blog.csdn.net/liukuan73/article/details/78710496

https://tonybai.com/2017/07/20/fix-cannot-access-dashboard-in-k8s-1-6-4/
https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/


curl --cacert /etc/kubernetes/pki/ca.crt --cert /etc/kubernetes/pki/front-proxy-ca.crt --key /etc/kubernetes/pki/front-proxy-ca.key \
  --resolve api.wardle.svc:6443:10.102.112.152 -v \
  https://api.wardle.svc/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder

curl --cacert /etc/kubernetes/pki/ca.crt --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key  \
  --resolve api.wardle.svc:6443:10.102.112.152 -v \
  https://api.wardle.svc/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder



OK:

curl  --cacert /etc/kubernetes/pki/ca.crt --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key   https://192.168.178.137:6443/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder


curl  --cacert /etc/kubernetes/pki/ca.crt --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key   https://192.168.178.137:6443/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder
{
  "kind": "Flunder",
  "apiVersion": "wardle.k8s.io/v1alpha1",
  "metadata": {
    "name": "my-first-flunder",
    "namespace": "default",
    "selfLink": "/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder",
    "uid": "1d2d6fa2-5cc1-11e8-9ee9-0a580af40192",
    "resourceVersion": "2",
    "creationTimestamp": "2018-05-21T06:35:12Z",
    "labels": {
      "sample-label": "true"
    }
  },
  "spec": {
    
  },
  "status": {}
}

NG:
curl --cacert /etc/kubernetes/pki/ca.crt --cert /etc/kubernetes/pki/front-proxy-ca.crt --key /etc/kubernetes/pki/front-proxy-ca.key https://192.168.178.137:6443/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder

curl --cacert /etc/kubernetes/pki/ca.crt --cert /etc/kubernetes/pki/front-proxy-ca.crt --key /etc/kubernetes/pki/front-proxy-ca.key https://192.168.178.137:6443/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}

NG: 

``` shell
APISERVER=$(kubectl config view | grep server | cut -f 2- -d ":" | tr -d " ")
TOKEN=$(kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ')| grep -E '^token' | cut -f2 -d':' | tr -d '\t')
curl $APISERVER/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder --header "Authorization: Bearer $TOKEN" --insecure
```
```
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "flunders.wardle.k8s.io \"my-first-flunder\" is forbidden: User \"system:anonymous\" cannot get flunders.wardle.k8s.io in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "name": "my-first-flunder",
    "group": "wardle.k8s.io",
    "kind": "flunders"
  },
  "code": 403
}
```


解决一切ssl问题：



kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='^*$'



curl --request POST \
  --url 'https://YOUR_AUTH0_DOMAIN/users/USER_ID/impersonate' \
  --header 'authorization: Bearer YOUR_ACCESS_TOKEN' \
  --header 'content-type: application/json' \
  --data '{"protocol": "PROTOCOL_TO_USE","impersonator_id": "IMPERSONATOR_ID","client_id": "YOUR_CLIENT_ID","additionalParameters":{"response_type": "code","state": ""}}'

APISERVER=$(kubectl config view | grep server | cut -f 2- -d ":" | tr -d " ")
TOKEN=$(kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ')| grep -E '^token' | cut -f2 -d':' | tr -d '\t')
curl $APISERVER/apis/wardle.k8s.io/v1alpha1/namespaces/default/flunders/my-first-flunder --header "Authorization: Bearer $TOKEN" --insecure  --header 'Impersonate-User: kubernetes-admin' --header 'Impersonate-Group: system:masters'



