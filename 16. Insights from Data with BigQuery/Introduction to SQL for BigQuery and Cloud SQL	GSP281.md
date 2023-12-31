
# GSP215 - HTTP Load Balancer with Cloud Armor

## Task 1. Configure HTTP and health check firewall rules

1. Buat firewall tcp/80

   ```bash
   gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
   ```

2. Buat health check

   ```bash
   gcloud compute firewall-rules create default-allow-health-check --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=http-server
   ```

## Task 2. Configure instance templates and create instance groups

1. Buat instance template

   ```bash
   gcloud compute instance-templates create us-central1 --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=default --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh,enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=68995227540-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=us-central1 --tags=http-server --create-disk=auto-delete=yes,boot=yes,device-name=us-central1,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230509,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
   ```

   ```bash
   gcloud compute instance-templates create europe-west1-template --machine-type=e2-micro --network-interface=network-tier=PREMIUM,subnet=default --metadata=startup-script-url=gs://cloud-training/gcpnet/httplb/startup.sh,enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=68995227540-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --region=europe-west1 --tags=http-server --create-disk=auto-delete=yes,boot=yes,device-name=us-central1,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230509,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --reservation-affinity=any
   ```

2. Buat instance group

   - us-central1

     ```bash
     gcloud beta compute instance-groups managed create us-central1-mig --base-instance-name=us-central1-mig --size=1 --template=us-central1 --zones=us-central1-c,us-central1-f,us-central1-b --target-distribution-shape=EVEN --instance-redistribution-type=PROACTIVE --list-managed-instances-results=PAGELESS --no-force-update-on-repair

     gcloud beta compute instance-groups managed set-autoscaling us-central1-mig --region=us-central1 --cool-down-period=45 --max-num-replicas=2 --min-num-replicas=1 --mode=on --target-cpu-utilization=0.8
     ```

   - europe-west1

     ```bash
     gcloud beta compute instance-groups managed create europe-west1-mig --project=qwiklabs-gcp-01-d5693c3493f3 --base-instance-name=europe-west1-mig --size=1 --template=europe-west1-template --zones=europe-west1-b,europe-west1-d,europe-west1-c --target-distribution-shape=EVEN --instance-redistribution-type=PROACTIVE --list-managed-instances-results=PAGELESS --no-force-update-on-repair
     ```

     ```bash
     gcloud beta compute instance-groups managed set-autoscaling europe-west1-mig --project=qwiklabs-gcp-01-d5693c3493f3 --region=europe-west1 --cool-down-period=45 --max-num-replicas=2 --min-num-replicas=1 --mode=on --target-cpu-utilization=0.8
     ```

## Task 3. Configure the HTTP Load Balancer

## Task 4. Test the HTTP Load Balancer

1. Buat vm instances

   ```bash
   gcloud compute instances create siege-vm --zone=us-central1-a --machine-type=e2-medium --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=68995227540-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=siege-vm,image=projects/debian-cloud/global/images/debian-11-bullseye-v20230509,mode=rw,size=10,type=projects/qwiklabs-gcp-01-d5693c3493f3/zones/us-central1-c/diskTypes/pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any
   ```

2. Connect SSH

   ```bash
   sudo apt-get -y install siege
   ```

3. Siege

   ```bash
   siege -c 150 -t120s http://$LB_IP
   ```

## Task 5. Denylist the siege-vm
