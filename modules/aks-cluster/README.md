#kubernetes_ingress

Ingress is a collection of rules that allow inbound connections to reach the endpoints defined by a backend. 
An Ingress can be configured to give services externally-reachable urls, load balance traffic, terminate SSL, offer name based virtual hosting etc.
``` yml
resource "kubernetes_service" "example" {
  metadata {
    name = "ingress-service"
  }
  spec {
    port {
      port = 80
      target_port = 80
      protocol = "TCP"
    }
    type = "NodePort"
  }
}

resource "kubernetes_ingress" "example" {
  wait_for_load_balancer = true
  metadata {
    name = "example"
    annotations = {
      "kubernetes.io/ingress.class" = "nginx"
    }
  }
  spec {
    rule {
      http {
        path {
          path = "/*"
          backend {
            service_name = kubernetes_service.example.metadata.0.name
            service_port = 80
          }
        }
      }
    }
  }
}

# Display load balancer hostname (typically present in AWS)
output "load_balancer_hostname" {
  value = kubernetes_ingress.example.status.0.load_balancer.0.ingress.0.hostname
}

# Display load balancer IP 
output "load_balancer_ip" {
  value = kubernetes_ingress.example.status.0.load_balancer.0.ingress.0.ip
}
```
