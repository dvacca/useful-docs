# Cloud Run - Step by step

Here you have how to deploy a website in a docker (using Flask in this example) on [Cloud Round](https://cloud.google.com/run) in a few clicks with multi-branch continuous build and deployment in place.

1. Create a project on [GitHub](github.com) (no need to be public)

2. Enable API and add required permissions on Google cloud (also billing needs to be enabled for project)

   1. [Enable needed APIs](https://console.cloud.google.com/flows/enableapi?apiid=cloudbuild.googleapis.com,run.googleapis.com,containerregistry.googleapis.com,cloudresourcemanager.googleapis.com)

   2. [Enable Cloud Run Admin role](https://console.cloud.google.com/cloud-build/settings)

3. Create a trigger on your Git repository in [Cloud Build](https://console.cloud.google.com/cloud-build/builds):
   - select a _name_ of preference
   - in source you need to connect and authenticate to your GitHub repository (Cloud Build GitHub App)
   - leave all default values

4. Add in your repository files below:

   1. `cloudbuild.yaml`
      ```yaml
      substitutions:

           # Adapt values below according to your region and project id
           _REGION:       your-preferred-region
           _PROJECT_ID:   your-project-id

           # Using lowercase branch name as image and service name for simplicity
           _IMAGE:        ${BRANCH_NAME,,}
           _SERVICE_NAME: ${BRANCH_NAME,,}

      steps:

      # Build the container image
      - name: 'gcr.io/cloud-builders/docker'
        args: ['build', '-t', 'gcr.io/${_PROJECT_ID}/${_IMAGE}', '.']

      # Push the container image to Container Registry
      - name: 'gcr.io/cloud-builders/docker'
        args: ['push', 'gcr.io/${_PROJECT_ID}/${_IMAGE}']

      # Deploy container image to Cloud Run and make the service available to everyone
      - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
      entrypoint: gcloud
        args: ['run', 'deploy', '${_SERVICE_NAME}', '--image', 'gcr.io/${_PROJECT_ID}/${_IMAGE}',
               '--region', '${_REGION}', '--platform', 'managed', '--allow-unauthenticated',
               '--update-env-vars', 'NAME=${_SERVICE_NAME}']

      images:
        - gcr.io/${_PROJECT_ID}/${_IMAGE}
      ```

   2. [main.py](https://github.com/GoogleCloudPlatform/python-docs-samples/blob/master/run/helloworld/main.py) to create a basic hello world Flask website

   3. [Dockerfile](https://github.com/GoogleCloudPlatform/python-docs-samples/blob/master/run/helloworld/Dockerfile) to build your image

   4. [.dockerignore](https://github.com/GoogleCloudPlatform/python-docs-samples/blob/master/run/helloworld/.dockerignore) to skip some files from docker image

5. Check last build status on [Cloud Build](https://console.cloud.google.com/cloud-build/builds)/Build History by clicking on the build Id. If build is green, current service URL is going to be reported in build logs just after `Service URL:`.

6. On [Cloud Run](https://console.cloud.google.com/run?authuser=3&project=cloud-run-test-bis) a service should have been created with the same branch name.

7. (Optional) There are two ways of assigning a custom (owned) domain, by clicking on _Manage custom domains_ and then:

   1. mapping an external verified domain : map a service as subdomain by adding a `CNAME` field in external DNS server

   2. using [Cloud Domains API](https://console.cloud.google.com/marketplace/product/google/domains.googleapis.com)

Any merge on git up will automatically trigger a new build and deployment.

## References

- [Continuous deployment from Git using Cloud Build](https://cloud.google.com/run/docs/continuous-deployment-with-cloud-build)

- [Using payload bindings and bash-style string operations in substitutions](https://cloud.google.com/cloud-build/docs/configuring-builds/use-bash-and-bindings-in-substitutions)

- [Quickstart: Build and Deploy](https://cloud.google.com/run/docs/quickstarts/build-and-deploy)

- [Mapping custom domains](https://cloud.google.com/run/docs/mapping-custom-domains)

- [Pricing](https://cloud.google.com/run/pricing)