# Kubeit Complex example

Demonstrate to deploy applications with different configs per

* environment

* region

* namespace

* use kubeit charts

# Configuration

1. Source folder: **chart** as entry provided during onboarding
   It is simple chart with helm dependency to `kubeit-app-of-apps chart`.

   Values contains definition of applications to deploy:

   ```yaml
   ---
   applications: &apps
     # name of application
     - name: standard-workload
       # node pool to use: linux or windows
       containerOS: linux
       chart:
         # folder on the same repo where configurations are stored
         path: deploy
       environments:
         # define environments to deploy application
         # name of environment: dev, nonprod, prod
         - name: dev
           # namespaces used on dev environment
           namespaces:
             - standard
             - standard-test
           # regions to deploy in dev environment
           # for nonprod/prod: eastus2, westeurope, southeastasia
           region:
             - westeurope
           # cluster colour: blue, green or cluster name
           clusterColour:
             - blue
             - dev001

   # Keep as it
   global:
     chart:
       ignoreMissingValueFiles: true
       skipCrds: true
     # reference applications as YAML anchors to do not copy-paste
     # passing values to helm subchart is only thru global dictionary
     applications: *apps
   ```

   It would create ArgoCD CR applications per env, per region, per namespace which match cluster settings, e.g. if cluster is dev, configuration for dev environment would be applied.

2. Source folder **deploy** contains simple chart with helm dependency to `kubeit-deployment-chart` and application(s) configuration.

   It should have **appValues** folder to override values for `kubeit-deployment-chart`.
   Folder structure:

   ```
   ├── deploy
   │   ├── appValues
   │   │   ├── global.yaml # global values for all apps in this folder
   │   │   ├── <app name>
   │   │   │   ├── appGlobal.yaml # global values for application
   │   │   │   ├── environment
   │   │   │   │   ├── <env name> # specific environment configs
   │   │   │   │   │   ├── <env name>.yaml # environment config
   │   │   │   │   ├── region
   │   │   │   │   │   ├── <region name>.yaml # specific regional config
   │   │   │   │   │   ├── cluster
   │   │   │   │   │   │   ├── <colour>.yaml
   │   │   │   │   │   │   ├── namespaces
   │   │   │   │   │   │   │   ├── <namespace name>.yaml # specific settings per namespace
   ```

   # Development environment

3. IDE for coding: Eclipse or Visual Studio Code

4. git

5. [pre-commit](https://pre-commit.com/)

# Workshop tasks

1. Change namespaces to deploy your application:

      * `chart/values.yaml` at application dictionary

      * `deploy/appValues/standard-workload/environment/dev/region/westeurope/cluster/namespaces/\<rename to new namespace>`

2. Do changes in existing application configuration, e.g. increase limits.

3. Deploy another application using **deploy** folder

4. Deploy another application using different folder

# References:

1. [KubeIT docs](https://docs.kubeit.dnv.com)

2. https://github.com/argoproj/argocd-example-apps

3. https://helm.sh/docs/chart_template_guide/subcharts_and_globals/
