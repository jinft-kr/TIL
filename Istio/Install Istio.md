# [Install with Istioctl](https://istio.io/latest/docs/setup/install/istioctl/)

### Prerequisites
- [Download the Istio release](https://istio.io/latest/docs/setup/getting-started/#download)
  - `curl -L https://istio.io/downloadIstio | sh -`
  - `cd istio-1.16.1`
  - `export PATH=$PWD/bin:$PATH`

### Install Istio
- `istioctl install --set profile=demo -y`
- `kubectl label namespace default istio-injection=enabled`

### If you want to bind elastic Ip in Ingress gateway(Load Balancer) and create daemonset for istio ingress
- `istioctl install --set profile=minimal -y`
  - `istiod` 만 설치
    - istio-ingressgateway 와 istio-engressgateway를 같이 설치하게 되면 loadbalancer 의 IP가 자동 생성 되고 변경이 어려움
    - 따라서 istiod만 설치하여 istio-ingressgateway는 직접 설치하고 직접 설치시에 생성한 IP를 연결해준다.
- `terraform apply`
  - Elastic IP 생성
  - Kubernetes Service type loadbalancer 에 elastic ip 연결
  - Istio ingress DaemonSet 생성
  - Kubernetes role binding

### Terraform script
```
# Node that these are not Istio Gateway's component, but similar to Services.
# Pods under these configuration will forward traffic.

resource "google_compute_address" "istio_ingress_endpoint_ip" {
  # To be used for external Network Load Balancer, google_compute_address should be used instead of google_compute_global_address.
  # google_compute_global_address is only valid when we want to create external Http(s) Load Balancer.
  name = "istio-ingress-endpoint-ip"
  address_type = "EXTERNAL"
}

resource "kubernetes_namespace" "istio_ingress" {
  metadata {
    name = local.ingress_namespace

    labels = {
      istio-injection = "enabled"
    }
  }
}

resource "kubernetes_service" "istio_ingress" {
  metadata {
    name = local.ingress_service_name
    namespace = kubernetes_namespace.istio_ingress.metadata[0].name

    annotations = {
      # For every service in K8S, below annotation shoud be set and kept.
      "cloud.google.com/neg" = "{\"ingress\" : true }"
    }
    labels = local.ingress_labels
  }
  spec {
    type = "LoadBalancer"
    selector = local.ingress_labels
    port {
      port = 80
      name = "http"
    }
    port {
      port = 443
      name = "https"
    }
    port {
      port = 15021
      name = "status-port"
    }

    load_balancer_ip = google_compute_address.istio_ingress_endpoint_ip.address

    external_traffic_policy = "Local"
    internal_traffic_policy = "Cluster"
  }
}

resource "kubernetes_daemonset" "istio_ingress" {
  metadata {
    name = local.ingress_daemonset_name
    namespace = kubernetes_namespace.istio_ingress.metadata[0].name

    labels = local.ingress_labels
  }
  spec {
    selector {
      match_labels = local.ingress_labels
    }
    template {
      metadata {
        annotations = {
          # Select the gateway injection template (rather than the default sidecar template)
          "inject.istio.io/templates" = "gateway"
        }
        labels = merge(local.ingress_labels, {
          # Enable gateway injection. If connecting to a revisioned control plane, replace with "istio.io/rev: revision-name"
          "sidecar.istio.io/inject" = "true"
        })
      }

      spec {
        service_account_name = "default"

        container {
          name = "istio-proxy"
          image = "auto" # The image will automatically update each time the pod starts.
        }
      }
    }
  }
}

resource "kubernetes_role" "istio_ingress" {
  metadata {
    name = "istio-ingressgateway-sds"
    namespace = kubernetes_namespace.istio_ingress.metadata[0].name
  }
  rule {
    api_groups = [
      ""
    ]
    resources  = [
      "secrets"
    ]
    verbs      = [
      "get",
      "watch",
      "list"
    ]
  }
}


resource "kubernetes_role_binding" "istio_ingress" {
  metadata {
    name = "istio-ingressgateway-sds-binding"
    namespace = kubernetes_namespace.istio_ingress.metadata[0].name
  }
  role_ref {
    api_group = "rbac.authorization.k8s.io"
    kind      = "Role"
    name      = kubernetes_role.istio_ingress.metadata[0].name
  }
  subject {
    kind = "ServiceAccount"
    name = "default" # This will be attached to Ingress Gateway Pods.
    namespace = kubernetes_namespace.istio_ingress.metadata[0].name
  }
}
```