# GCP-CloudRun
Deploying to Google Cloud Run with Terraform

# gcp-microservices-iac



## Getting started

To make it easy for you to get started with GitLab, here's a list of recommended next steps.

Already a pro? Just edit this README.md and make it your own. Want to make it easy? [Use the template at the bottom](#editing-this-readme)!

## Add your files

- [ ] [Create](https://docs.gitlab.com/ee/user/project/repository/web_editor.html#create-a-file) or [upload](https://docs.gitlab.com/ee/user/project/repository/web_editor.html#upload-a-file) files
- [ ] [Add files using the command line](https://docs.gitlab.com/ee/gitlab-basics/add-file.html#add-a-file-using-the-command-line) or push an existing Git repository with the following command:

```
cd existing_repo
git remote add origin https://gitlab.com/574n13y/gcp-microservices-iac.git
git branch -M main
git push -uf origin main
```

## Integrate with your tools

- [ ] [Set up project integrations](https://gitlab.com/574n13y/gcp-microservices-iac/-/settings/integrations)

## Collaborate with your team

- [ ] [Invite team members and collaborators](https://docs.gitlab.com/ee/user/project/members/)
- [ ] [Create a new merge request](https://docs.gitlab.com/ee/user/project/merge_requests/creating_merge_requests.html)
- [ ] [Automatically close issues from merge requests](https://docs.gitlab.com/ee/user/project/issues/managing_issues.html#closing-issues-automatically)
- [ ] [Enable merge request approvals](https://docs.gitlab.com/ee/user/project/merge_requests/approvals/)
- [ ] [Set auto-merge](https://docs.gitlab.com/ee/user/project/merge_requests/merge_when_pipeline_succeeds.html)

## Test and Deploy

Use the built-in continuous integration in GitLab.

- [ ] [Get started with GitLab CI/CD](https://docs.gitlab.com/ee/ci/quick_start/index.html)
- [ ] [Analyze your code for known vulnerabilities with Static Application Security Testing (SAST)](https://docs.gitlab.com/ee/user/application_security/sast/)
- [ ] [Deploy to Kubernetes, Amazon EC2, or Amazon ECS using Auto Deploy](https://docs.gitlab.com/ee/topics/autodevops/requirements.html)
- [ ] [Use pull-based deployments for improved Kubernetes management](https://docs.gitlab.com/ee/user/clusters/agent/)
- [ ] [Set up protected environments](https://docs.gitlab.com/ee/ci/environments/protected_environments.html)

***

## Project status
 ### Prerequisites
To follow this tutorial you will need:
 - Terraform CLI. I recommend using the latest version, currently v0.14. Instructions to download and install Terraform can be found here.
 - Google Cloud SDK. The most recent version should also work well for this tutorial. Installation instructions here.
 - A Google Cloud account. If you don’t have one, create it here.

### Initial setup
   - Start by authenticating the SDK to Google Cloud:
     ![Screenshot 2024-01-27 150727](https://github.com/574n13y/GCP-CloudRun/assets/35293085/578e42da-5966-4e47-a56a-a273a8f137fb)

   - Create a new project where your Cloud Run service will be deployed. Replace PROJECT_ID and PROJECT_NAME with the desired values:
     ![Screenshot 2024-01-27 150804](https://github.com/574n13y/GCP-CloudRun/assets/35293085/53246714-e794-466c-b93d-ac1df8e92188)

   - Creating your first service
     ```
     terraform {
       required_version = ">= 0.14"

       required_providers {
        # Cloud Run support was added on 3.3.0
           google = ">= 3.3"
         }
       }

       provider "google" {
        # Replace `PROJECT_ID` with your project
        project = "vivesh-405513"
       }

       resource "google_project_service" "run_api" {
        service = "run.googleapis.com"

         disable_on_destroy = true
        }

        resource "google_cloud_run_service" "run_service" {
         name = "app"
         location = "us-central1"

     template {
    spec {
      containers {
         image = "gcr.io/google-samples/hello-app:1.0"
      }
    }
      }

     traffic {
    percent         = 100
    latest_revision = true
      }

     # Waits for the Cloud Run API to be enabled
     depends_on = [google_project_service.run_api]
    }

      resource "google_cloud_run_service_iam_member" "run_all_users" {
      service  = google_cloud_run_service.run_service.name
      location = google_cloud_run_service.run_service.location
      role     = "roles/run.invoker"
      member   = "allUsers"
      }

           resource "google_storage_bucket" "auto-expire" {
              name          = "stanley_bucket_iac"
            location      = "US"
            force_destroy = true

        public_access_prevention = "enforced"
        }

       output "service_url" {
      value = google_cloud_run_service.run_service.status[0].url
      }
     ```
     
   - Let’s stop for a while and check what the code above is doing:
     ```
    name: the name of your service. It will be displayed in the public URL.
    location: the region where your service will run. See all the options here.
    image: The Docker image that will be used to create the container. Cloud Run has direct support for images from the Container Registry and Artifact Registry.
    traffic: controls the traffic for this revision. The percent property indicates how much traffic will be redirected to this revision. latest_revision specifies that this traffic configuration needs to be used for the latest revision.
    depends_on: waits for a resource to be ready, in this case, the Cloud Run API.
    ```
   - Invoking the service --> By default, Cloud Run services are private and secured by IAM. To access them, you would need valid credentials with at least the Cloud Run Invoker permission set.
   - Deploying the infrastructure
     `` terraform init ``
     ![Screenshot 2024-01-27 150837](https://github.com/574n13y/GCP-CloudRun/assets/35293085/6a57c05b-1588-44ff-b2a8-5871a7d4ed9d)

     `` terraform plan ``
     ![Screenshot 2024-01-27 150901](https://github.com/574n13y/GCP-CloudRun/assets/35293085/cfbdae3a-16d7-42fe-a1b3-9e3d269562f4)

     `` terrafrom apply ``
     ![Screenshot 2024-01-27 150926](https://github.com/574n13y/GCP-CloudRun/assets/35293085/bd3de2df-1129-424a-8686-55747762862c)

     ![Screenshot 2024-01-27 151008](https://github.com/574n13y/GCP-CloudRun/assets/35293085/301cbf70-301c-4f7d-ac22-f9f2a2b82ecc)

     ![Screenshot 2024-01-27 151029](https://github.com/574n13y/GCP-CloudRun/assets/35293085/3c7bf48d-0836-4692-9176-24f5610f4d64)

   - Updating the service ``image = "gcr.io/google-samples/hello-app:2.0" ``
     ![Screenshot 2024-01-27 151054](https://github.com/574n13y/GCP-CloudRun/assets/35293085/fb444c46-5b7e-4267-926d-7612538884b6)

   - Run terraform apply to deploy the changes:
     ![Screenshot 2024-01-27 151120](https://github.com/574n13y/GCP-CloudRun/assets/35293085/4441ff7f-ed15-4167-b0b2-5fd650cf9cb0)
     
     ![Screenshot 2024-01-27 151144](https://github.com/574n13y/GCP-CloudRun/assets/35293085/c9396f14-ab9c-403f-8b28-83e2aa623a4e)
     
     ![Screenshot 2024-01-27 151205](https://github.com/574n13y/GCP-CloudRun/assets/35293085/93cb97f2-29b1-42db-aad1-3f2b846b7bb2)


   - Cleaning up
   - To delete all resources created with Terraform, run the following command and confirm the prompt:
     ![Screenshot 2024-01-27 151304](https://github.com/574n13y/GCP-CloudRun/assets/35293085/99f225a1-c4b0-4c50-be80-1c0d8693f794)
     
     ![Screenshot 2024-01-27 151333](https://github.com/574n13y/GCP-CloudRun/assets/35293085/03e77c44-0041-4806-8186-a23c71e68d30)

     ![Screenshot 2024-01-27 151359](https://github.com/574n13y/GCP-CloudRun/assets/35293085/cf733c1b-c699-4251-b7af-cee121800bdf)


   - This will disable the Cloud Run API, delete the Cloud Run service and its permissions.
     ![Screenshot 2024-01-27 151447](https://github.com/574n13y/GCP-CloudRun/assets/35293085/f0e3e46c-ecf2-4621-996d-27a695d7f879)

   - The project was created using the gcloud CLI tool, so you will need to delete it manually. For that, you can run:
     ![Screenshot 2024-01-27 151624](https://github.com/574n13y/GCP-CloudRun/assets/35293085/e2466325-51f5-4496-99d2-abf114446800)

## Gitlab 
  ![Screenshot 2024-01-27 135055](https://github.com/574n13y/GCP-CloudRun/assets/35293085/9d771e63-f76b-4a46-91b5-329118d0a52d)
  
  ![Screenshot 2024-01-27 141054](https://github.com/574n13y/GCP-CloudRun/assets/35293085/ecbbedad-7195-44c6-820a-9dbfce0fe536)

  ![Screenshot 2024-01-27 152701](https://github.com/574n13y/GCP-CloudRun/assets/35293085/1b0d3ddc-e19c-4f7b-9284-a21dda59aa4e)


