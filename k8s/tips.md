# tips

## 创建命令的简称

``` shell

#!/bin/bash
export PATH=/u01/DockerTools:$PATH
export PATH=/harbor:$PATH
#source /u01/DockerImages/var.sh
alias di='docker images'
alias de='docker exec -it'
alias dr='docker run -it'
alias di='docker images'
alias ds='docker ps'
alias dri='docker rmi'
alias drs='docker rm'
alias kg='kubectl get '
alias kga='kubectl get all '
alias kd=' kubectl describe '
alias kgaa='kubectl get all --all-namespaces'
alias kl='kubectl logs '
alias kc='kubectl create '
alias ka='kubectl apply '
alias kr='kubectl delete '


```