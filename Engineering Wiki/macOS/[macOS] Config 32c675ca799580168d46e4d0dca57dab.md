# [macOS] Config

Owner: Nam Tran
Last edited time: March 23, 2026 6:30 PM

```bash
if type brew &>/dev/null; then
    FPATH=$(brew --prefix)/share/zsh-completions:$FPATH

    autoload -Uz compinit
    compinit
  fi

source <(kubectl completion zsh)
source <(oc completion zsh)
source <(helm completion zsh)

# Định nghĩa đường dẫn file riêng
export K8S_CONFIG="$HOME/.kube/config-k8s"
export OCP1_CONFIG="$HOME/.kube/config-ocp1"
export OCP2_CONFIG="$HOME/.kube/config-ocp2"
 
# Tạo Alias thông minh
# alias kube="KUBECONFIG=$K8S_CONFIG kubectl"
# alias oc1="KUBECONFIG=$OCP1_CONFIG oc"
# alias oc2="KUBECONFIG=$OCP2_CONFIG oc"
 
# Hàm chuyển đổi nhanh môi trường cho tab hiện tại
set-k8s() { export KUBECONFIG=$K8S_CONFIG; echo "Context: K8s VVT"; }
set-ocp1() { export KUBECONFIG=$OCP1_CONFIG; echo "Context: OpenShift VVT1"; }
set-ocp2() { export KUBECONFIG=$OCP2_CONFIG; echo "Context: OpenShift VVT2"; }
```