# 🚀 Innovatech Solutions - Arquitectura de Microservicios EKS

Este repositorio contiene la infraestructura y el código de despliegue para el sistema de microservicios de Innovatech Solutions. El proyecto está construido sobre **Java/Kotlin con Spring Boot** y diseñado para operar en un entorno de alta disponibilidad utilizando **Amazon Elastic Kubernetes Service (EKS)**.

---

## 🏗️ Arquitectura de la Solución

El sistema sigue una arquitectura orientada a microservicios, desplegada en la nube de AWS bajo los siguientes lineamientos:

*   **VPC (Virtual Private Cloud):** Red privada virtual configurada con subredes públicas y privadas (en las zonas `us-east-1a` y `us-east-1b`) para garantizar la alta disponibilidad y el aislamiento seguro de los recursos.
*   **EC2 (Elastic Compute Cloud):** Los nodos de trabajo (Worker Nodes) del clúster utilizan instancias tipo `t3.medium`, proporcionando el balance ideal entre cómputo y memoria para la ejecución de contenedores Java.
*   **ECR (Elastic Container Registry):** Actúa como repositorio seguro y privado para las imágenes Docker generadas en cada compilación. Cada imagen es etiquetada (tagged) de manera única según el commit.
*   **EKS (Elastic Kubernetes Service):** Orquestador central que administra el ciclo de vida de los contenedores, garantizando que los servicios estén siempre disponibles.
*   **IAM (Identity and Access Management):** Gestión estricta de permisos mediante roles (ej. `LabRole`) para permitir la comunicación segura entre EC2, EKS y ECR sin exponer credenciales estáticas.

---

## ⚙️ CI/CD: Pipeline de Despliegue (GitHub Actions)

La integración y el despliegue continuo están completamente automatizados a través de GitHub Actions. El flujo se ejecuta exclusivamente al realizar un *push* a la rama obligatoria **`deploy`**.

El pipeline (`deploy.yml`) consta de las siguientes etapas automatizadas:
1.  **Build:** Compilación del código fuente y ejecución de pruebas unitarias.
2.  **Docker Build (Multietapa):** Creación de una imagen Docker optimizada utilizando un enfoque *multi-stage*. Se emplea una imagen base ligera (`alpine`/`slim`) para minimizar el peso final de la imagen en producción.
3.  **Push a ECR:** Autenticación segura mediante *Secrets* de GitHub y publicación de la imagen versionada en Amazon ECR.
4.  **Deploy a EKS:** Actualización automática de los manifiestos de Kubernetes para desplegar la nueva versión de la imagen en los nodos del clúster EKS.

---

## 📈 Configuración del Clúster y Autoescalado (HPA)

Para garantizar la estabilidad del sistema bajo picos de tráfico, el clúster cuenta con mecanismos de escalabilidad y distribución de carga:

### Balanceo de Carga
Se utiliza el **AWS Load Balancer Controller** para exponer los servicios al exterior. Esto distribuye eficientemente el tráfico HTTP/HTTPS entrante entre los distintos *Pods* activos, evitando la saturación de una sola instancia.

### Justificación del Horizontal Pod Autoscaler (HPA)
El clúster implementa **HPA** para escalar dinámicamente el número de *Pods* en función de la demanda. 
*   **Métrica de Escalamiento:** El HPA está configurado para monitorear el consumo de **CPU y Memoria** de los contenedores.
*   **Justificación Técnica:** Los microservicios desarrollados en Spring Boot (Java/Kotlin) pueden presentar picos de consumo de memoria y CPU durante el procesamiento de solicitudes concurrentes. El HPA garantiza que, si un servicio supera el 70% de uso de CPU, Kubernetes despliegue automáticamente réplicas adicionales (*Scale Out*). Una vez que la carga disminuye, el sistema reduce las réplicas (*Scale In*), optimizando el uso de recursos y reduciendo los costos operativos de Innovatech Solutions.

---

## 🔍 Monitoreo, Logs y Métricas

La observabilidad es un pilar fundamental de esta arquitectura:
*   **Logs del Clúster:** Los registros de inicialización y ejecución (tanto del *Control Plane* como de los *Worker Nodes*) pueden ser auditados directamente desde la terminal mediante `kubectl logs`, permitiendo trazar el ciclo de vida completo de cada petición.
*   **Métricas de Salud:** Se implementan *Liveness* y *Readiness Probes* en los manifiestos de Kubernetes. Esto asegura que el balanceador de carga solo envíe tráfico a los *Pods* que han iniciado correctamente y están listos para recibir peticiones, garantizando un *uptime* continuo.

---
*Desarrollado y desplegado para la evaluación de arquitectura Cloud.*
