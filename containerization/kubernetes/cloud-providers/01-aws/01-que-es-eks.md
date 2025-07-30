# ☁️ ¿Qué es EKS (Elastic Kubernetes Service)?

EKS (Amazon Elastic Kubernetes Service) es el servicio gestionado de Kubernetes de Amazon Web Services (AWS). Permite desplegar, operar y escalar clústeres de Kubernetes sin necesidad de gestionar manualmente el plano de control.

AWS se encarga de mantener el plano de control altamente disponible, seguro y escalable, mientras tú administras los nodos y las aplicaciones que se ejecutan en el clúster.

---

## 🎯 Objetivo de EKS

Simplificar la ejecución de aplicaciones en contenedores sobre Kubernetes, integrando la potencia de AWS con las capacidades de orquestación de Kubernetes.

---

## 🧱 Componentes clave de EKS

| Componente                  | ¿Gestionado por EKS?     |
|----------------------------|--------------------------|
| Plano de control (control plane) | ✅ Sí                 |
| Nodos worker (EC2, Fargate)      | ✅ Manual o automático |
| Escalamiento de nodos           | ✅ Opcional (Auto Scaling Groups) |
| Seguridad y autenticación       | ✅ IAM + RBAC integrados |
| Monitoreo                      | ✅ Integración con CloudWatch |

---

## 🚀 Opciones de ejecución

EKS permite dos formas de correr workloads:

### 1. **EC2 (Amazon Elastic Compute Cloud)**
- Tú gestionas los nodos de trabajo (instancias EC2).
- Flexibilidad total sobre el entorno.

### 2. **AWS Fargate (Serverless)**
- AWS gestiona completamente la infraestructura.
- Solo defines recursos de los pods.

---

## 🖼️ Arquitectura simplificada

```plaintext
+-----------------------------+
|     AWS Management Console |
+-----------------------------+
              |
              v
     +---------------------+
     |  EKS Control Plane  |
     | (K8s API Server, etcd) |
     +---------------------+
              |
       +-------------+
       | Node Groups |
       +-------------+
        /           \
   [EC2 Nodes]   [Fargate Pods]
````

---

## 🧰 Integraciones destacadas

* 🔐 IAM para control de acceso (IAM for Service Accounts).
* 📦 ECR (Elastic Container Registry) para imágenes de contenedores.
* 📈 CloudWatch para métricas y logs.
* 🌐 ELB/NLB para servicios expuestos al exterior.
* 🔄 CodePipeline y CodeBuild para CI/CD.

---

## ✅ Ventajas de EKS

* 🌍 Alta disponibilidad (multi-AZ).
* 🛡️ Seguridad con IAM, VPC y roles.
* 🤝 Compatibilidad con herramientas estándar de Kubernetes.
* 📈 Escalado automático de nodos y pods.
* 💼 Integración con servicios enterprise de AWS.

---

## 📦 Casos de uso típicos

* Microservicios en contenedores.
* Cargas de trabajo empresariales migradas desde on-premise.
* Plataformas SaaS multi-tenant.
* Aplicaciones distribuidas que requieren resiliencia.

---

## ⚠️ Consideraciones

* 💰 El plano de control tiene un costo fijo por hora.
* ⚙️ Algunas configuraciones requieren conocimientos avanzados de red y seguridad en AWS (VPC, IAM, SGs).
* 🧠 Requiere dominio de CLI (`eksctl` o `aws eks`) para una buena automatización.

---

## 🔗 Recursos adicionales

* [Documentación oficial de EKS](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
* [EKS vs ECS vs Fargate](https://aws.amazon.com/compare/the-difference-between-amazon-ecs-amazon-eks-and-aws-fargate/)
* [Tutorial: Primer clúster con `eksctl`](https://eksworkshop.com)

---

## 📚 Siguiente paso

➡️ Revisa el archivo `02-deploy-cluster-eks.md` para desplegar tu primer clúster usando `eksctl` o la consola de AWS.

