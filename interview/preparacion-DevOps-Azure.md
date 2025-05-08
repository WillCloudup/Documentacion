# 💼 Preparación para Entrevista: DevOps Engineer con Azure

Este documento es un compendio de **preguntas y respuestas clave** que tuve que prerarparme para entrevistas técnicas centradas en **Azure DevOps, AKS, WAF, App Services, IaC, observabilidad, API Management, y más**.

> ✅ *Contiene prácticas reales, soluciones a incidentes, arquitectura recomendada y preguntas estilo entrevista técnica.*

---

## 🔹 CI/CD y Despliegues en Azure

### ❓¿Qué servicios usas para implementar CI/CD en Azure?

**Respuesta:** Azure DevOps (Pipelines), GitHub Actions, Azure Container Registry (ACR), Azure Kubernetes Service (AKS), Azure App Services. Para integraciones legacy: Jenkins.

### ❓¿Cómo integras Jenkins con Azure?

**Respuesta:** Con Azure Service Principal para autenticación, plugins de Azure en Jenkins y ejecución de jobs vía CLI o REST API. Uso de credenciales almacenadas como secrets.

### ❓¿Cómo haces despliegues blue/green en AKS?

**Respuesta:** A través de Ingress Controllers (NGINX) con reglas de routing, Helm y anotaciones de tráfico, o mediante Istio con Canary/BlueGreen. Validación previa con probes.

### ❓¿Cómo haces despliegues blue/green en AKS?

**Respuesta:** A través de Ingress Controllers (NGINX) con reglas de routing, Helm y anotaciones de tráfico, o mediante Istio con Canary/BlueGreen. Validación previa con probes.

### ❓¿Qué diferencia hay entre Azure App Services y AKS?

**Respuesta:**

* App Services es PaaS, gestionado, ideal para apps web y APIs con menos complejidad operativa.
* AKS es IaaS orientado a contenedores, ideal para microservicios complejos con alto control y escalabilidad.
---

## 🔹 Azure Kubernetes Service (AKS)

### ❓¿Cómo administras AKS en producción?

**Respuesta:** Uso de:

* `kubectl`, `k9s` para debugging.
* Ingress Controllers con IP pública.
* Helm para gestión de releases.
* Terraform para IaC y HPA para autoscaling.
* Alertas con Prometheus + Grafana / Azure Monitor.

### ❓¿Qué es el HPA y cómo se activa?

**Respuesta:** Horizontal Pod Autoscaler, se basa en CPU, RAM u otros métricas. Configurado con requests y limits en los pods y métricas expuestas con metrics-server.

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
```

### ❓¿Cómo determinas el mínimo necesario para activar autoscaling?

**Respuesta:** Se basa en `resource requests`. Si CPU o memoria superan el umbral (ej. 80%), se escala. Necesario que el nodo tenga capacidad.

### ❓¿Cómo gestionar múltiples entornos (dev, qa, prod) en AKS?

**Respuesta:**

* Múltiples namespaces por entorno
* Contextos separados en kubeconfig
* RBAC segmentado
* Uso de Workload Identity para acceso seguro a otros servicios

### ❓¿Cómo manejar upgrades del cluster?

**Respuesta:**

* Usar nodepools separados para rolling upgrade
* Validar en dev y staging antes de producción
* `az aks upgrade` controlado por versión compatible

### ❓¿Cómo resolverías fallos de readiness probes?

**Respuesta:**

* Validar puerto o endpoint configurado en el pod
* Revisar logs (`kubectl logs`) y eventos (`kubectl describe pod`)
* Ajustar `initialDelaySeconds` o `timeoutSeconds` si es necesario

---

## 🔹 Observabilidad y Diagnóstico

### ❓¿Qué herramientas usas para observabilidad?

**Respuesta:**

* Azure Monitor / Log Analytics
* Grafana + Prometheus en AKS
* Application Insights
* Fluent Bit / Dapr para logging estructurado
* Dashboards personalizados

### ❓¿Cómo resuelves un error 500 en un flujo Front Door > APIM > AKS?

**Diagnóstico:**

1. Revisar Application Gateway y WAF logs.
2. Verificar salud del backend en APIM.
3. Usar `kubectl logs` y `kubectl describe` en pods AKS.
4. Validar la conexión desde Front Door.

### ❓¿Cómo implementar trazabilidad distribuida?

**Respuesta:** Con OpenTelemetry o Azure Application Insights. Se propagan headers (`traceparent`, `request-id`) y se conectan servicios con correlación automática.

### ❓¿Qué métricas básicas debes monitorear en un clúster AKS?

**Respuesta:**

* CPU y memoria de nodos y pods
* Estado del nodo (`Ready`, `DiskPressure`)
* Tiempo de respuesta (latencia)
* Disponibilidad de endpoints
* Errores 4xx/5xx


---

## 🔹 WAF en Azure Front Door

### ❓¿Cómo habilitar WAF en Azure Front Door?

**Respuesta:**

1. Crear política WAF (Global Front Door)
2. Activar OWASP 3.2 en modo Prevention
3. Asignar reglas personalizadas (IPs, headers, paths)
4. Asignar política a Frontend Domain de Front Door

### ❓¿Qué protecciones básicas debes activar?

* OWASP Core Ruleset
* Rate Limiting
* Header Inspection
* Geoblocking

### ❓¿Cómo asegurar un endpoint en Azure APIM?

**Respuesta:**

* OAuth2 o JWT Bearer Tokens
* Validación de claims en políticas
* Reglas de IP o headers (firewall lógico)
* Seguridad en rutas y métodos HTTP

### ❓¿Cómo ocultar información sensible en logs?

**Respuesta:**

* Usar Application Insights con filtros de telemetry
* Redactar campos sensibles con políticas en APIM (`<set-header>` oculto)
* Usar variables de entorno seguras (`Key Vault`)
---

## 🔹 API Management y Seguridad

### ❓¿Qué políticas aplicarías para restringir el acceso a una API?

**Respuesta:**

* Validación de JWT con claims.
* IP filtering por subnet.
* Rate limit por suscripción y usuario.
* Reglas personalizadas con política `<check-header>`, `<rate-limit-by-key>`.

### ❓¿Cómo aplicar Rate Limit por usuario?

**Respuesta:**

```xml
<rate-limit-by-key calls="10" renewal-period="60" counter-key="@(context.Request.Headers.GetValueOrDefault("x-user-id", "anonymous"))" />
```

### ❓¿Cómo manejas múltiples workspaces con Terraform?

**Respuesta:**

* Uso de `terraform workspace new/dev/prod`
* Separación lógica por entorno
* Backend en Azure Storage con locking

### ❓¿Qué módulo has reutilizado más en proyectos Azure?

**Respuesta:**

* AKS con RBAC y nodepools
* App Service con conexión a Key Vault
* Networking: VNet, Subnets, NSG

---

## 🔹 IaC y Terraform en Azure

### ❓¿Cómo modularizas tu código Terraform?

**Respuesta:** Uso de módulos por servicio (AKS, ACR, App Service). Variables, outputs y remote state backend en Azure Storage. Ejemplo:

```hcl
module "aks" {
  source = "./modules/aks"
  cluster_name = var.cluster_name
  node_count = 3
}
```

### ❓¿Qué es un Workspace en Terraform y cómo lo usas?

**Respuesta:** Permite aislar estados entre entornos (`dev`, `qa`, `prod`). Uso con `terraform workspace new` y backends remotos.

### ❓Cliente reporta error 200 sin contenido en API vía Front Door + APIM + AKS. ¿Cómo diagnosticar?

**Respuesta:**

1. Revisar el backend pool de APIM, estado y health check
2. Logs de NGINX Ingress Controller
3. Confirmar respuesta real desde pod con `curl` o `kubectl port-forward`
4. Validar rutas en policies de APIM y reglas de enrutamiento

### ❓¿Qué harías si un despliegue no sustituye correctamente una variable en el pipeline clásico?

**Respuesta:**

* Revisar definiciones en variables de Release
* Validar formato \$(variable) y stage scope
* Usar reemplazo manual en el archivo YAML con tokenización
---

## 🔹 Azure Networking y Balanceo

### ❓¿Cómo balanceas tráfico entre 2 APIs desde Front Door?

**Respuesta:**

* Usar Rules Engine para dividir tráfico por %.
* Configurar 2 rutas con `Backend Pools` distintos.
* Política de distribución aleatoria (Round Robin).

### ❓¿Qué otros servicios de traffic management puedes usar?

* Azure Load Balancer
* Application Gateway
* Azure Traffic Manager (DNS-based)
* API Management con rutas condicionales

---

## 🔹 Seguridad y Enmascaramiento

### ❓¿Cómo manejarías información sensible en logs o respuestas?

**Respuesta:**

* Uso de Application Gateway WAF para remover headers.
* Enmascaramiento vía policies en APIM (`set-body`, `set-header`).
* Uso de Azure Key Vault para guardar secrets.

---

## 🔹 Despliegues y Casos Reales

### ❓¿Qué errores de despliegue has tenido y cómo los solucionaste?

**Respuesta:**

* Error por build ID estático → solución: usar variables dinámicas.
* Timeout en pipeline → optimización de pasos pesados.
* Imagen de Docker corrupta → recrear build y push manualmente.

---

## 🔹 Integraciones y Automatización

### ❓¿Qué scripts en Python te han sido útiles?

* Script para reiniciar pods que fallan por imagen:

```python
import os
os.system("kubectl rollout restart deployment my-deploy")
```

* Verificar estado de Azure Resource:

```python
from azure.identity import DefaultAzureCredential
from azure.mgmt.resource import ResourceManagementClient
```

### ❓¿Qué patrones de arquitectura aplicas en Azure?

* Front Door + APIM + AKS
* App Services + Key Vault + App Config
* Event Grid + Functions + Logic Apps
* DataBricks + ADLS para pipelines de datos

---

## 🔹 Extra: Arquitectura y SRE

### ❓¿Qué patrón arquitectónico usarías para alta disponibilidad?

**Respuesta:** Active-Active multirregión, con Front Door como punto de entrada y backends balanceados.

### ❓¿Qué servicios usarías para distribución inteligente de tráfico?

**Respuesta:**

* Azure Front Door: Global, HTTP/HTTPS, WAF integrado
* Traffic Manager: DNS-based, por prioridad o geolocalización
* Application Gateway: L7, WAF, basado en rutas

---

## 🔍 Preguntas Basadas en Experiencia Real

### ❓¿Cómo solucionaste un fallo de despliegue en AKS por un `CrashLoopBackOff`?

**Respuesta:** Analicé los logs con `kubectl logs`, identifiqué que la variable `DB_HOST` estaba mal seteada desde el ConfigMap. Hice rollback al deployment anterior mientras corregía el valor. Usamos liveness probe y readiness probe para validar comportamiento correcto. Añadí validaciones de variables en la app y alertas proactivas.

---

### ❓¿Cómo implementaste blue/green deployment en una plataforma AKS?

**Respuesta:** Utilicé Helm con dos versiones del deployment (`v1` y `v2`) en diferentes namespaces. El Ingress Controller (NGINX) redirigía tráfico usando anotaciones específicas. Hicimos pruebas con smoke tests, y una vez validado, cambiamos el route del 100% del tráfico a la nueva versión. Luego eliminamos la anterior.

---

### ❓Cuéntame una ocasión donde una política de seguridad mal aplicada causó problemas

**Respuesta:** Una vez en Azure API Management se aplicó una política `<check-header>` obligatoria para `Authorization`, sin considerar que algunos endpoints públicos no requerían token. Esto bloqueó consultas legítimas. La solución fue aplicar la política sólo a rutas específicas usando `<when>` y `<choose>`.

---

### ❓¿Has tenido que automatizar despliegues en múltiples regiones?

**Respuesta:** Sí, para una plataforma distribuida usé Terraform y módulos reutilizables para crear recursos en `East US` y `West Europe`. Implementé un pipeline con Azure DevOps que ejecutaba por región, integrando variables de entorno y estados remotos independientes.

---

### ❓¿Cómo implementaste el monitoreo centralizado?

**Respuesta:** Integré Azure Monitor con Application Insights en todos los microservicios y contenedores del AKS. Habilitamos logs personalizados, traces y Live Metrics. Agregamos Azure Log Analytics para consultas avanzadas, y creamos dashboards con métricas clave (latencia, errores, throughput).

---

### ❓¿Has tenido una caída total en producción? ¿Qué hiciste?

**Respuesta:** Tuvimos una caída por fallo en el `nodepool` del AKS. Activé el scaling de otro pool mientras investigábamos. Reiniciamos servicios críticos en nuevo pool y luego restauramos el estado del cluster. A partir de eso, configuramos un pool de fallback y probes más estrictos.

---

### ❓¿Cómo lograste asegurar la exposición de APIs sensibles?

**Respuesta:** Implementé autenticación con Azure AD y OAuth2 en APIM, y además filtrado por IP. Configuramos WAF en Front Door para bloquear patrones de ataque. Sensitive headers fueron removidos o redirigidos con políticas personalizadas de APIM (`<set-header>` y `<remove-header>`).

---

### ❓¿Has trabajado con Workload Identity en AKS?

**Respuesta:** Sí, implementé Workload Identity para permitir que pods accedieran a Key Vault sin necesidad de secrets o service principals. Configuré la federación OIDC, la identity federada y las roles asignadas directamente en Azure RBAC. Esto mejoró la seguridad notablemente.

---

### ❓¿Cómo gestionaste un problema con escalado automático que no respondía?

**Respuesta:** Notamos que el HPA estaba definido con thresholds bajos, pero el Cluster Autoscaler no escalaba nodos. Investigamos y vimos que los nodos tenían recursos insuficientes para nuevos pods. Ajustamos `resourceRequests` y `limits`, y luego escaló correctamente.

---

### ❓¿Has tenido problemas de latencia en APIs?

**Respuesta:** Sí, al exponer APIs en App Gateway y APIM. Identificamos latencia generada por backends lentos, enrutamiento ineficiente y logs síncronos en el backend. Cacheamos respuestas comunes en APIM y optimizamos llamadas internas con Azure Functions y cola asíncrona.

---

### ❓¿Cómo documentas tus implementaciones?

**Respuesta:** Uso Markdown versionado en GitHub, incluyendo diagramas de arquitectura (con draw\.io o mermaid.js), flujos de CI/CD, decisiones técnicas y scripts. También generamos runbooks para operaciones manuales temporales.

---

### ❓¿Cómo hiciste troubleshooting de un error intermitente 500/200 desde Front Door?

**Respuesta:** Revisamos logs de Front Door, Application Gateway y APIM. Los 500 eran generados por errores en pods que fallaban en ciertos headers específicos. Añadimos logging más detallado en el backend y ajustamos el modelo de error handling en la app. Activamos Health Probes mejorados para aislar nodos inestables.

---

### ❓¿Cómo protegiste endpoints internos en AKS sin exponerlos a internet?

**Respuesta:** Usamos Azure Private Link con DNS privado, y Service Endpoints en subnets específicas. Aplicamos Network Policies en Kubernetes para que sólo servicios autorizados accedieran a esos pods. También usamos Internal Load Balancer (`internal: true`).

---

## 📌 RECOMENDACIONES PARA ENTREVISTA

* Lleva claro cómo aplicas prácticas SRE en Azure (dashboards, alertas, SLA).
* Habla de tus pipelines de CI/CD como flujos claros: lint → test → build → push → deploy → verify.
* Describe problemas que resolviste en producción con logs y métricas.
* Ten a mano ejemplos de código (Terraform, Python, YAML).
* Refuerza tu experiencia tanto en backend como en frontend (static web apps, React, etc).

---

William Quintero 
DevOps Engineer
GitHub Hero🚀
