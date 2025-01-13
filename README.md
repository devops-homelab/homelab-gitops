
# Repository Overview

This repository contains the Helm chart definitions and infrastructure configurations for the MediTrack project. It includes charts for various microservices, infrastructure components, and local environment configurations.

## Repository Structure

```
.
├── README.md
└── charts
    ├── app
    │   └── meditrack
    │       ├── mt-aggregator-service
    │       │   ├── Chart.yaml
    │       │   ├── templates
    │       │   │   └── cronjob.yaml
    │       │   └── values
    │       │       └── values.yaml
    │       ├── mt-appointment-scheduling-service
    │       │   ├── Chart.yaml
    │       │   ├── templates
    │       │   │   ├── ingress.yaml
    │       │   │   ├── rollout.yaml
    │       │   │   └── service.yaml
    │       │   └── values
    │       │       └── values.yaml
    │       ├── mt-doctor-record-service
    │       │   ├── Chart.yaml
    │       │   ├── templates
    │       │   │   ├── ingress.yaml
    │       │   │   ├── rollout.yaml
    │       │   │   └── service.yaml
    │       │   └── values
    │       │       └── values.yaml
    │       ├── mt-notification-service
    │       │   ├── Chart.yaml
    │       │   ├── templates
    │       │   │   ├── ingress.yaml
    │       │   │   ├── rollout.yaml
    │       │   │   └── service.yaml
    │       │   └── values
    │       │       └── values.yaml
    │       └── mt-patient-record-service
    │           ├── Chart.yaml
    │           ├── templates
    │           │   ├── ingress.yaml
    │           │   ├── rollout.yaml
    │           │   └── service.yaml
    │           └── values
    │               └── values.yaml
    ├── infra
    │   └── meditrack
    │       └── crossplane
    │           ├── Chart.yaml
    │           ├── crossplane.yaml
    │           └── values.yaml
    └── local-use-only
        ├── argocd-parent-app.yaml
        ├── argocd-values.yaml
        ├── db-creds.yaml
        ├── dockerhubcreds.yaml
        └── redshift-creds.yaml
```

## Directory Descriptions

### `charts/app/meditrack`
Contains Helm charts for MediTrack microservices:
- **`mt-aggregator-service`**: Handles data aggregation tasks.
  - `cronjob.yaml`: Defines a scheduled job for data aggregation.
- **`mt-appointment-scheduling-service`**: Manages appointment scheduling.
  - `ingress.yaml`, `rollout.yaml`, `service.yaml`: Define Kubernetes resources.
- **`mt-doctor-record-service`**: Handles doctor record management.
- **`mt-notification-service`**: Sends notifications to users.
- **`mt-patient-record-service`**: Manages patient records.

Each service contains:
- `Chart.yaml`: Metadata about the chart.
- `templates/`: Kubernetes resource templates.
- `values/values.yaml`: Default configurations.

### `charts/infra/meditrack/crossplane`
Contains Helm chart for infrastructure management using Crossplane:
- `crossplane.yaml`: Crossplane resource definitions.
- `values.yaml`: Configuration values for Crossplane resources.

### `local-use-only`
Files for local development and testing:
- `argocd-parent-app.yaml`: Parent application definition for ArgoCD.
- `argocd-values.yaml`: Configuration values for ArgoCD.
- `db-creds.yaml`: Database credentials.
- `dockerhubcreds.yaml`: Docker Hub credentials.
- `redshift-creds.yaml`: AWS Redshift credentials.

## Contributing
Feel free to open issues or submit pull requests to improve the repository.

## License
This project is licensed under the MIT License. See `LICENSE` for more details.
