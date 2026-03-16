# Proyecto Final - Tax Calculator con Pipeline CI/CD

Proyecto que integra una aplicación web de calculadora de impuestos con un pipeline de CI/CD basado en **Tekton** para automatizar la construcción, pruebas y despliegue de imágenes Docker.

## Estructura del Proyecto

```
git-ProyectoFinal/
├── juphc-tc_pipeline/     # Pipeline Tekton CI/CD
│   ├── pipeline.yaml     # Definición del pipeline
│   ├── tasks.yaml        # Tareas personalizadas (npm, jasmine)
│   └── pvc.yaml          # PersistentVolumeClaim para el pipeline
│
└── zntwk-tax_calculator/ # Aplicación calculadora de impuestos
    ├── index.html
    ├── script.js
    ├── style.css
    ├── taxCalculator.js
    ├── Dockerfile
    └── package.json
```

## Componentes

### Tax Calculator (`zntwk-tax_calculator`)

Aplicación web que calcula impuestos según tramos de ingresos. Incluye:

- **Tecnologías**: HTML, CSS, JavaScript, Bootstrap 5
- **Tests**: Jasmine para pruebas unitarias
- **Contenedor**: Imagen Docker basada en Nginx para servir los archivos estáticos

#### Ejecución local

```bash
cd zntwk-tax_calculator
npm install
npm test          # Ejecutar tests con Jasmine
```

#### Construcción Docker

```bash
docker build -t tax-calculator .
docker run -p 8080:80 tax-calculator
```

### Pipeline Tekton (`juphc-tc_pipeline`)

Pipeline de CI/CD que automatiza:

1. **Clone** — Clona el repositorio del código fuente
2. **npm install** — Instala dependencias de Node.js
3. **Tests** — Ejecuta pruebas unitarias con Jasmine
4. **Build** — Construye la imagen Docker con Buildah y la sube a IBM Cloud Container Registry

#### Requisitos previos

- Cluster de Kubernetes con Tekton instalado
- ClusterTask `git-clone` disponible
- ClusterTask `buildah` para construcción de imágenes
- StorageClass `skills-network-learner` (o adaptar `pvc.yaml`)

#### Despliegue del pipeline

```bash
# Aplicar recursos
kubectl apply -f juphc-tc_pipeline/pvc.yaml
kubectl apply -f juphc-tc_pipeline/tasks.yaml
kubectl apply -f juphc-tc_pipeline/pipeline.yaml
```

#### Ejecutar el pipeline

Crear un `PipelineRun` que especifique:

- URL del repositorio a clonar
- Workspace con el PVC
- Parámetro `app-name` para el nombre de la imagen

La imagen se construye como: `us.icr.io/<namespace>/<app-name>`

## Flujo del Pipeline

```
┌─────────┐    ┌─────────────┐    ┌────────┐    ┌────────┐
│  Clone  │───▶│ npm install │───▶│ Tests  │───▶│ Build  │
└─────────┘    └─────────────┘    └────────┘    └────────┘
     │                 │                │             │
     │                 │                │             └── Buildah → IBM Cloud Registry
     │                 │                └── Jasmine
     │                 └── npm install
     └── git-clone
```

## Licencia

[Apache 2.0](juphc-tc_pipeline/LICENSE)
