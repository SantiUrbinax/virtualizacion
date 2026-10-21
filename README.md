# Assessment 06 â€” ImplementaciÃ³n de GitOps con Argo CD

**Estudiante:** Angel Santiago Urbina Lopez  
**Curso:** VirtualizaciÃ³n  
**Repositorio:** [SantiUrbinax/virtualizacion](https://github.com/SantiUrbinax/virtualizacion)  
**Rama de evaluaciÃ³n:** `assessment-06`

## 1. IntroducciÃ³n

Esta prÃ¡ctica consistiÃ³ en implementar un flujo de despliegue continuo siguiendo el enfoque **GitOps** mediante Argo CD, sobre un clÃºster local de Kubernetes gestionado con Minikube.

Se reutilizÃ³ la infraestructura desarrollada anteriormente: cuatro aplicaciones web NGINX, el controlador de entrada Traefik y MetalLB como proveedor de direcciones para servicios `LoadBalancer`. A partir de los archivos YAML almacenados en GitHub, se crearon dos recursos `Application` que administran los servicios existentes en el namespace `parcial-asul`.

Para comprobar el proceso, se cambiÃ³ el nÃºmero de rÃ©plicas de **Portal** de una a dos en el repositorio Git. Argo CD detectÃ³ la nueva revisiÃ³n, sincronizÃ³ el estado declarado y Kubernetes mostrÃ³ las dos rÃ©plicas disponibles sin necesidad de aplicar manualmente el Deployment.

## 2. Objetivos

### Objetivo general

Implementar Argo CD en Kubernetes para administrar las aplicaciones mediante GitOps, empleando GitHub como fuente de verdad de la configuraciÃ³n declarativa.

### Objetivos especÃ­ficos

- Instalar Argo CD en el namespace independiente `argocd-asul`.
- Exponer su interfaz web a travÃ©s de Traefik y el dominio local `argocd.asul.local`.
- Conectar Argo CD con el repositorio de GitHub, rama `assessment-06`.
- Crear dos `Application` que administren los recursos de Portal, CatÃ¡logo, Noticias y Soporte.
- Verificar que ambas aplicaciones alcancen los estados `Synced` y `Healthy`.
- Modificar un manifiesto en GitHub y comprobar que el cambio se refleje en Kubernetes.
- Conservar los manifiestos de infraestructura como cÃ³digo y las evidencias del procedimiento.

## 3. TecnologÃ­as utilizadas

| Herramienta | VersiÃ³n o configuraciÃ³n | FunciÃ³n |
|---|---|---|
| Kubernetes | v1.37.0 (entorno de actividades anteriores) | OrquestaciÃ³n de contenedores |
| Minikube | Driver Docker | ClÃºster local de Kubernetes |
| Docker Desktop | Windows 11 | Entorno de contenedores |
| `kubectl` | CLI de Kubernetes | AdministraciÃ³n y validaciÃ³n |
| Kustomize | Integrado mediante `kubectl -k` | PersonalizaciÃ³n de manifiestos de Argo CD |
| Argo CD | Manifiesto oficial `stable` (versiÃ³n no fijada) | SincronizaciÃ³n GitOps |
| Traefik | Ingress Controller | Enrutamiento HTTP |
| MetalLB | `192.168.49.240` | DirecciÃ³n del LoadBalancer de Traefik |
| GitHub | Rama `assessment-06` | Repositorio de manifiestos |
| PowerShell | Windows 11 | EjecuciÃ³n de comandos |

> **Nota de reproducibilidad:** La configuraciÃ³n de instalaciÃ³n de Argo CD utiliza el manifiesto oficial referenciado mediante `stable`. Para despliegues repetibles conviene sustituirlo por una versiÃ³n especÃ­fica validada.

## 4. Arquitectura GitOps

```text
               GitHub
        rama assessment-06
                  |
                  v
             Argo CD
         namespace argocd-asul
                  |
        +---------+----------+
        |                    |
        v                    v
     asul-web          asul-servicios
        |                    |
  Portal + CatÃ¡logo    Noticias + Soporte
        |                    |
        +---------+----------+
                  |
                  v
       Kubernetes / Minikube
        namespace parcial-asul
                  |
          Services e Ingress
                  |
                Traefik
```

GitHub almacena el estado deseado. Argo CD compara ese estado con los recursos del clÃºster y, mediante la polÃ­tica de sincronizaciÃ³n automÃ¡tica, aplica las diferencias detectadas. Los recursos web conservan sus `ConfigMap`, `Deployment`, `Service` e `Ingress`.

## 5. PreparaciÃ³n de la rama Git

La rama solicitada fue creada desde `main` y publicada en el mismo repositorio utilizado para las actividades anteriores:

```powershell
git fetch origin
git switch main
git pull --ff-only origin main
git switch -c assessment-06
git push -u origin assessment-06
```

Repositorio: <https://github.com/SantiUrbinax/virtualizacion>.

Los manifiestos de la evaluaciÃ³n anterior se recuperaron en la nueva rama sin modificar su historial:

```powershell
git restore --source=assessment-02 -- apps metallb traefik
```

Posteriormente se reorganizaron los recursos para que cada `Application` administrara un conjunto distinto: `apps/asul-web/recursos.yaml` para Portal y CatÃ¡logo, y `apps/asul-servicios/recursos.yaml` para Noticias y Soporte.

## 6. Estructura del proyecto

```text
virtualizacion/
â”œâ”€â”€ README.md
â”œâ”€â”€ apps/
â”‚   â”œâ”€â”€ asul-web/
â”‚   â”‚   â””â”€â”€ recursos.yaml
â”‚   â”œâ”€â”€ asul-servicios/
â”‚   â”‚   â””â”€â”€ recursos.yaml
â”‚   â”œâ”€â”€ servicios-web.yaml          # Manifiestos originales de referencia
â”‚   â””â”€â”€ rutas-ingress.yaml          # Ingress originales de referencia
â”œâ”€â”€ argocd/
â”‚   â”œâ”€â”€ kustomization.yaml
â”‚   â”œâ”€â”€ config-http.yaml
â”‚   â”œâ”€â”€ ingress.yaml
â”‚   â””â”€â”€ applications/
â”‚       â””â”€â”€ applications.yaml
â”œâ”€â”€ metallb/
â”‚   â””â”€â”€ pool-red.yaml
â”œâ”€â”€ traefik/
â”‚   â””â”€â”€ configuracion-traefik.yaml
â””â”€â”€ img/
    â”œâ”€â”€ 04-argocd-dominio.png
    â”œâ”€â”€ 05-applications-antes.png
    â”œâ”€â”€ 06-github-cambio.png
    â”œâ”€â”€ 07-applications-despues.png
    â”œâ”€â”€ 08-portal-replicas.png
    â””â”€â”€ 09-commit-sincronizado.png
```

> Los archivos de `img/` son los nombres previstos para las capturas de esta evaluaciÃ³n. Deben incorporarse al repositorio para que las imÃ¡genes se visualicen en GitHub.

## 7. CreaciÃ³n del namespace

Se creÃ³ un namespace dedicado a Argo CD:

```powershell
kubectl create namespace argocd-asul


![Namespace de Argo CD](img/01-argocd-namespace.png)
```

La creaciÃ³n respondiÃ³:

```text
namespace/argocd-asul created
```

Para verificarlo:

```powershell
kubectl get namespace argocd-asul
```

## 8. InstalaciÃ³n de Argo CD

### 8.1. ConfiguraciÃ³n con Kustomize

Se creÃ³ `argocd/kustomization.yaml` para utilizar los manifiestos oficiales y adaptar las referencias de permisos a `argocd-asul`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: argocd-asul

resources:
  - https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

patches:
  - path: config-http.yaml
  - target:
      kind: ClusterRoleBinding
    patch: |-
      - op: replace
        path: /subjects/0/namespace
        value: argocd-asul
```

Para habilitar la comunicaciÃ³n HTTP interna detrÃ¡s de Traefik se creÃ³ `argocd/config-http.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cmd-params-cm
data:
  server.insecure: "true"
```

Esta modalidad HTTP es especÃ­fica de este laboratorio local; en entornos expuestos debe usarse HTTPS y una configuraciÃ³n de seguridad adecuada.

### 8.2. ValidaciÃ³n e instalaciÃ³n

```powershell
kubectl kustomize argocd > $null
kubectl apply --server-side --force-conflicts -k argocd
```

Se crearon los CRD, ServiceAccounts, roles y vinculaciones, ConfigMaps, Secrets, Services, Deployments y el StatefulSet del controlador.

### 8.3. VerificaciÃ³n de los componentes

```powershell
kubectl get pods -n argocd-asul


![Pods de Argo CD en estado Running](img/02-argocd-pods.png)
kubectl rollout status deployment/argocd-server -n argocd-asul --timeout=180s
```

**Resultado:** los siete pods principales alcanzaron el estado `1/1 Running` y el Deployment `argocd-server` informÃ³ `successfully rolled out`.

**Evidencia sugerida:** captura de la lista de pods de Argo CD.

## 9. ExposiciÃ³n de Argo CD mediante Traefik

### 9.1. ConfiguraciÃ³n del Ingress

Se creÃ³ `argocd/ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-ingress
  namespace: argocd-asul
spec:
  ingressClassName: traefik
  rules:
    - host: argocd.asul.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: argocd-server
                port:
                  number: 80
```

Se aplicÃ³ y verificÃ³ con:

```powershell
kubectl apply -f argocd/ingress.yaml
kubectl get ingress -n argocd-asul


![Ingress de Argo CD mediante Traefik](img/03-argocd-ingress.png)
```

**Resultado observado:**

```text
NAME             CLASS     HOSTS               ADDRESS          PORTS
argocd-ingress   traefik   argocd.asul.local   192.168.49.240   80
```

### 9.2. LimitaciÃ³n de red de Windows y acceso local

Aunque MetalLB asignÃ³ `192.168.49.240` a Traefik, las pruebas `Test-NetConnection` desde Windows hacia esa IP y hacia el nodo `192.168.49.2` fallaron. Tampoco se encontrÃ³ una ruta especÃ­fica desde Windows hacia la red `192.168.49.0/24`.

Por ello se utilizÃ³ el siguiente acceso alternativo, que conserva a Traefik en la ruta de las solicitudes:

```powershell
kubectl port-forward -n traefik svc/traefik 8081:80
```

En `C:\Windows\System32\drivers\etc\hosts` se configurÃ³:

```text
127.0.0.1 argocd.asul.local
```

La direcciÃ³n utilizada en el navegador fue:

**http://argocd.asul.local:8081**

> **LimitaciÃ³n respecto al requisito:** el Ingress anuncia la IP de MetalLB `192.168.49.240`, pero en esta modalidad el archivo `hosts` apunta a `127.0.0.1` y se requiere mantener abierto `kubectl port-forward`. Se verificÃ³ el acceso por dominio a travÃ©s de Traefik, **no** el acceso directo de Windows a la IP del LoadBalancer.

### 9.3. ComprobaciÃ³n HTTP

Se ejecutÃ³:

```powershell
curl.exe -I --max-time 10 -H "Host: argocd.asul.local" http://127.0.0.1:8081/
```

**Respuesta:** `HTTP/1.1 200 OK`.

La solicitud completa devolviÃ³ el HTML con el tÃ­tulo `<title>Argo CD</title>`, y la interfaz se abriÃ³ correctamente desde el navegador.

![Acceso a Argo CD mediante el dominio local y Traefik](img/04-argocd-dominio.png)

## 10. ConfiguraciÃ³n de las aplicaciones Kubernetes

Los cuatro servicios existentes utilizan NGINX y disponen de los siguientes recursos:

- **ConfigMap:** contenido HTML de cada sitio.
- **Deployment:** contenedor NGINX y cantidad de rÃ©plicas.
- **Service:** exposiciÃ³n interna dentro del clÃºster.
- **Ingress:** publicaciÃ³n HTTP mediante Traefik.

### 10.1. Grupo `asul-web`

Directorio: `apps/asul-web/`.

| Recurso | Portal | CatÃ¡logo |
|---|---|---|
| Deployment | `portal` | `catalogo` |
| Service | `portal-service` | `catalogo-service` |
| ConfigMap | `portal-html` | `catalogo-html` |
| Ingress | `portal-ingress` | `catalogo-ingress` |
| Dominio | `portal.asul.local` | `catalogo.asul.local` |

### 10.2. Grupo `asul-servicios`

Directorio: `apps/asul-servicios/`.

| Recurso | Noticias | Soporte |
|---|---|---|
| Deployment | `noticias` | `soporte` |
| Service | `noticias-service` | `soporte-service` |
| ConfigMap | `noticias-html` | `soporte-html` |
| Ingress | `noticias-ingress` | `soporte-ingress` |
| Dominio | `noticias.asul.local` | `soporte.asul.local` |

### 10.3. ValidaciÃ³n de manifiestos

```powershell
kubectl apply --dry-run=client -f apps/asul-web/recursos.yaml
kubectl apply --dry-run=client -f apps/asul-servicios/recursos.yaml
```

Ambas validaciones concluyeron sin errores: se verificaron **ocho recursos por grupo**, diecisÃ©is en total, sin modificar el estado del clÃºster.

## 11. ConfiguraciÃ³n GitOps de Argo CD

### 11.1. Repositorio y rama

Las dos `Application` estÃ¡n vinculadas al repositorio:

```text
https://github.com/SantiUrbinax/virtualizacion.git
```

Rama: `assessment-06`.

### 11.2. DefiniciÃ³n de `asul-web`

El archivo `argocd/applications/applications.yaml` contiene:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: asul-web
  namespace: argocd-asul
spec:
  project: default
  source:
    repoURL: https://github.com/SantiUrbinax/virtualizacion.git
    targetRevision: assessment-06
    path: apps/asul-web
  destination:
    server: https://kubernetes.default.svc
    namespace: parcial-asul
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### 11.3. DefiniciÃ³n de `asul-servicios`

En el mismo archivo se definiÃ³:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: asul-servicios
  namespace: argocd-asul
spec:
  project: default
  source:
    repoURL: https://github.com/SantiUrbinax/virtualizacion.git
    targetRevision: assessment-06
    path: apps/asul-servicios
  destination:
    server: https://kubernetes.default.svc
    namespace: parcial-asul
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

`automated` habilita la sincronizaciÃ³n automÃ¡tica; `selfHeal` reconcilia diferencias en el clÃºster, y `prune` permite eliminar recursos administrados que se retiren de Git. La opciÃ³n `CreateNamespace=true` permite crear el namespace destino cuando sea necesario.

### 11.4. AplicaciÃ³n y verificaciÃ³n

```powershell
kubectl apply --dry-run=client -f argocd/applications/applications.yaml
kubectl apply -f argocd/applications/applications.yaml
kubectl get applications -n argocd-asul
```

**Resultado inicial verificado:**

```text
NAME             SYNC STATUS   HEALTH STATUS
asul-servicios   Synced        Healthy
asul-web         Synced        Healthy
```

![Applications antes del cambio, en estado Synced y Healthy](img/05-applications-antes.png)

## 12. DemostraciÃ³n de sincronizaciÃ³n GitOps

### 12.1. Estado inicial

El Deployment `portal` tenÃ­a una rÃ©plica (`replicas: 1`). Antes del cambio, ambas Applications mostraban `Synced` y `Healthy`.

La revisiÃ³n sincronizada inicialmente por `asul-web` fue:

```text
e3d48418202a18e87233fc767a4fb8cb3974238b
```

### 12.2. ModificaciÃ³n del manifiesto

Se editÃ³ `apps/asul-web/recursos.yaml`, cambiando solo el nÃºmero de rÃ©plicas del Deployment `portal`:

```diff
-  replicas: 1
+  replicas: 2
```

`git --no-pager diff -- apps/asul-web/recursos.yaml` confirmÃ³ que no existÃ­an otros cambios en ese archivo.

### 12.3. Commit y publicaciÃ³n

```powershell
git add apps/asul-web/recursos.yaml
git commit -m "GitOps: aumentar replicas de portal de 1 a 2"
git push origin assessment-06
```

Commit publicado:

`f9cda54c31be15665a7abf8047d5f11ac148fee5`

[Consultar commit en GitHub](https://github.com/SantiUrbinax/virtualizacion/commit/f9cda54).

![Cambio de una a dos rÃ©plicas en GitHub](img/06-github-cambio.png)

### 12.4. ActualizaciÃ³n de la revisiÃ³n en Argo CD

Al consultar inmediatamente despuÃ©s del `push`, Argo CD todavÃ­a mostraba la revisiÃ³n anterior. Se solicitÃ³ una actualizaciÃ³n del repositorio:

```powershell
kubectl annotate application asul-web -n argocd-asul argocd.argoproj.io/refresh=hard --overwrite
```

Esta instrucciÃ³n solicitÃ³ una nueva consulta de la fuente Git, **sin aplicar manualmente el Deployment**.

Se verificÃ³ la revisiÃ³n reconocida por Argo CD:

```powershell
kubectl get application asul-web -n argocd-asul -o jsonpath="{.status.sync.revision}"
```

**Resultado:**

```text
f9cda54c31be15665a7abf8047d5f11ac148fee5
```

### 12.5. VerificaciÃ³n en Kubernetes

```powershell
kubectl get deployment portal -n parcial-asul
kubectl get pods -n parcial-asul -l app=portal
```

Se obtuvo:

```text
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
portal   2/2     2            2           16d
```

Los dos pods de Portal aparecieron como `1/1 Running`, uno preexistente y otro creado despuÃ©s del cambio.

![VerificaciÃ³n de las dos rÃ©plicas de Portal](img/08-portal-replicas.png)

### 12.6. Estado final de las Applications

```powershell
kubectl get applications -n argocd-asul
```

**Resultado:**

```text
NAME             SYNC STATUS   HEALTH STATUS
asul-servicios   Synced        Healthy
asul-web         Synced        Healthy
```

![Applications despuÃ©s del cambio, en estado Synced y Healthy](img/07-applications-despues.png)

La coincidencia del commit publicado y la revisiÃ³n sincronizada, junto con el estado de las rÃ©plicas, confirma que Argo CD aplicÃ³ el cambio a partir de GitHub.

![ConfirmaciÃ³n de la revisiÃ³n sincronizada por Argo CD](img/09-commit-sincronizado.png)

## 13. Resultados obtenidos

| Requisito o verificaciÃ³n | Resultado |
|---|---|
| Namespace independiente | `argocd-asul` |
| InstalaciÃ³n de Argo CD | Kustomize y manifiesto oficial `stable` |
| Pods principales | 7/7 `Running` |
| Ingress de Traefik | Creado y operativo |
| IP anunciada por el Ingress | `192.168.49.240` |
| Acceso web comprobado | `http://argocd.asul.local:8081` mediante port-forward a Traefik |
| Acceso directo al LoadBalancer desde Windows | **No conseguido** |
| Repositorio y rama | `SantiUrbinax/virtualizacion`, `assessment-06` |
| Applications | `asul-web`, `asul-servicios` |
| Estado inicial | Ambas `Synced / Healthy` |
| Cambio GitOps | Portal de 1 a 2 rÃ©plicas |
| Estado posterior | Ambas `Synced / Healthy` |
| Commit sincronizado | `f9cda54` |

## 14. Conclusiones

La prÃ¡ctica permitiÃ³ implementar Argo CD sobre Kubernetes y aprovechar la infraestructura desarrollada previamente, sin reconstruir los servicios web existentes.

La organizaciÃ³n de los manifiestos en dos grupos independientes facilitÃ³ la administraciÃ³n declarativa de Portal, CatÃ¡logo, Noticias y Soporte. Argo CD mantuvo ambas Applications en estado `Synced` y `Healthy`.

La prueba principal consistiÃ³ en modificar el nÃºmero de rÃ©plicas de Portal en GitHub. Tras actualizar la revisiÃ³n consultada por Argo CD, Kubernetes pasÃ³ de una a dos rÃ©plicas sin ejecutar `kubectl apply` manualmente sobre el Deployment. La coincidencia entre el commit publicado y la revisiÃ³n sincronizada corroborÃ³ el flujo GitOps.

En cuanto a la exposiciÃ³n web, se confirmÃ³ la comunicaciÃ³n entre Traefik y Argo CD mediante HTTP 200 y acceso desde el navegador con un dominio local. No obstante, el aislamiento de la red de Minikube en Docker Desktop impidiÃ³ acceder directamente a la IP de MetalLB desde Windows, por lo que se documentÃ³ el uso de `port-forward` como alternativa y la limitaciÃ³n de cumplimiento correspondiente.

## 15. Referencias

- [Argo CD â€” DocumentaciÃ³n oficial](https://argo-cd.readthedocs.io/en/stable/)
- [Argo CD â€” InstalaciÃ³n](https://argo-cd.readthedocs.io/en/stable/operator-manual/installation/)
- [Kubernetes â€” DocumentaciÃ³n](https://kubernetes.io/docs/)
- [Minikube â€” DocumentaciÃ³n](https://minikube.sigs.k8s.io/docs/)
- [Kustomize â€” DocumentaciÃ³n de Kubernetes](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)
- [Traefik â€” Kubernetes Ingress](https://doc.traefik.io/traefik/routing/providers/kubernetes-ingress/)
- [MetalLB â€” DocumentaciÃ³n](https://metallb.io/)
- [GitHub â€” Repositorio de la evaluaciÃ³n](https://github.com/SantiUrbinax/virtualizacion/tree/assessment-06)

