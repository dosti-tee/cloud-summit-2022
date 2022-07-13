# Deploy Sample Applications on GKE Autopilot
This repository consist of sample application yaml configurations to test the utilization of GKE Autopilot mode.

## Guestbook App
To run this application, obtain your cluster credentials as needed and then execute the below command inside the guestbook directory:
```
kubectl apply -f .
```

## Pubsub App for HPA Test
The configure in the pubsub folder is to test horizontal scalability of pods in th autopilot mode. To do this, follow the next steps:

1. Create a pubsub topic and subscription in your project with the names echo and echo-read respectively.
2. Create a service account for the stackdriver adapter to authenticate to GCP platform. Replace <<PROJECT_ID>> with your actual project id in the yaml config files [adapter](pubsub/deployment/adapter.yaml) & [pubsub-sa](pubsub/deployment/pubsub-sa.yaml), and commands below. (Note: workload identity access control is being used in autopilot mode.)
```
gcloud iam service-accounts create custom-metrics-sd-adapter --project <<PROJECT_ID>>

gcloud projects add-iam-policy-binding <<PROJECT_ID>> \
  --role "roles/monitoring.editor"
  --member "serviceAccount:custom-metrics-sd-adapter@<<PROJECT_ID>>.iam.gserviceaccount.com"

gcloud iam service-accounts add-iam-policy-binding \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:<<PROJECT_ID>>.svc.id.goog[custom-metrics/custom-metrics-stackdriver-adapter]" \
  custom-metrics-sd-adapter@<<PROJECT_ID>>.iam.gserviceaccount.com --project <<PROJECT_ID>>
```
3. Check if the stackdriver service account's permission works:
```
kubectl get --raw '/apis/external.metrics.k8s.io/v1beta1/namespaces/pubsub/pubsub.googleapis.com|subscription|num_undelivered_messages' | jq -C
```
4. Create a service account for your pubsub application:
```
gcloud iam service-accounts create pubsub-sa --project <<PROJECT_ID>>

gcloud projects add-iam-policy-binding <<PROJECT_ID>> \
  --role roles/pubsub.editor
  --member "serviceAccount:pubsub-sa@<<PROJECT_ID>>.iam.gserviceaccount.com"

gcloud iam service-accounts add-iam-policy-binding \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:<<PROJECT_ID>>.svc.id.goog[pubsub/pubsub-ksa]" \
  pubsub-sa@<<PROJECT_ID>>.iam.gserviceaccount.com --project <<PROJECT_ID>>
```
5. Change directory into the [pubsub/deployment](pubsub/deployment) folder. Execute the below command and use kubectl to verify that your deployment, hpa and other components are working.
```
kubectl apply -f .
```
6. Generate some messages on the pubsub run
```
for i in {1..100}; do gcloud pubsub topics publish echo --project <<PROJECT_ID>> -—message=“Autoscaling demo #${i}”; done
```

