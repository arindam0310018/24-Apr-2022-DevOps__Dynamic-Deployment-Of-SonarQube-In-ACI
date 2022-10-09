# DYNAMIC DEPLOYMENT OF SONARQUBE IN AZ CONTAINER INSTANCE USING DEVOPS

Greetings my fellow Technology Advocates and Specialists.

In this Session, I will demonstrate how to __Dynamically Deploy SONARQUBE__ in __AZURE CONTAINER INSTANCE__ using __AZURE DEVOPS PIPELINES__

| __LIVE RECORDED SESSION:-__ |
| --------- |
| __LIVE DEMO__ was Recorded as part of my Presentation in __CLOUD LUNCH AND LEARN__ Forum/Platform |
| Duration of My Demo = __44 Mins 19 Secs__ |
| [![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/0D5gURnxdz8/0.jpg)](https://www.youtube.com/watch?v=0D5gURnxdz8) |

| __REQUIREMENTS:-__ |
| --------- |

1. Azure Subscription
2. Azure DevOps Organisation and Project 
3. Azure Resource Manager Service Connection

| __Below follows the contents of the YAML File (Azure DevOps):-__ |
| --------- |

```
trigger:
  none

######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Please Provide the Subscription ID:-
  type: object
  default: 210e66cb-55cf-424e-8daa-6cad804ab604

- name: ServiceConnection
  displayName: Please Provide the Service Connection Name:-
  default: amcloud-cicd-service-connection
  values:
  - amcloud-cicd-service-connection

- name: RGName
  displayName: Please Provide the Name of the Resource Group:-
  type: object
  default: DemoRGSonarQube

- name: RGLocation
  displayName: Please Provide Location of Resource Group:-
  default: WestEurope
  values:
  - WestEurope

- name: validateIfRGExists
  displayName: Does the provided Resource Group Exists or do we create ?
  default: Existing
  values:
  - Existing
  - New

- name: ACIName
  displayName: Please Provide the Name of the Azure Container Instance:-
  type: object
  default: am-poc-sonarqube-aci

######################
#DECLARE VARIABLES:-
######################
variables:
  Artifact: AM
  cpu: 2 
  memory: 3.5
  port: 9000
  
#########################
# Declare Build Agents:-
#########################
pool:
  vmImage: windows-latest

###################
# Declare Stages:-
###################
stages:

- stage: VALIDATE_SELECTION_EXISTING
  condition: |
     and(eq('${{ parameters.validateIfRGExists }}', 'Existing'), 
       eq(variables['build.sourceBranch'], 'refs/heads/main')) 
  jobs:
  - job: IF_RG_EXISTS_DEPLOY_SONARQUBE 
    displayName: IF RG EXISTS DEPLOY SONARQUBE
    steps:
    - task: AzureCLI@2
      displayName: CHECK RG AND DEPLOY SONARQUBE
      inputs:
        azureSubscription: ${{ parameters.ServiceConnection }}
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show  
          $i = az group exists -n ${{ parameters.RGName }}
          if ($i -eq "true") {
          $j = az container list --query "[?contains(name, '${{ parameters.ACIName }}')].[provisioningState]" -o tsv
            if ($j -eq "Succeeded") {
            echo "##########################################################################################################################################"
            echo "${{ parameters.ACIName }} CONTAINER INSTANCE EXISTS IN THE ${{ parameters.RGName }} RESOURCE GROUP. CANNOT PROCEED WITH THE DEPLOYMENT"
            echo "##########################################################################################################################################"  
              }
            else {   
            az container create -g ${{ parameters.RGName }} --name ${{ parameters.ACIName }} --image sonarqube --ports $(port) --dns-name-label ${{ parameters.ACIName }} --cpu $(cpu) --memory $(memory)
              }
            }
          else {
          az group create -l ${{ parameters.RGLocation }} -n ${{ parameters.RGName }} 
          az container create -g ${{ parameters.RGName }} --name ${{ parameters.ACIName }} --image sonarqube --ports $(port) --dns-name-label ${{ parameters.ACIName }} --cpu $(cpu) --memory $(memory)    
            }

- stage: VALIDATE_SELECTION_NEW
  condition: |
     and(eq('${{ parameters.validateIfRGExists }}', 'New'), 
       eq(variables['build.sourceBranch'], 'refs/heads/main')) 
  jobs:
  - job: CREATE_RG_AND_DEPLOY_SONARQUBE 
    displayName: CREATE RG AND DEPLOY SONARQUBE
    steps:
    - task: AzureCLI@2
      displayName: CHECK RG AND DEPLOY SONARQUBE
      inputs:
        azureSubscription: ${{ parameters.ServiceConnection }}
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show  
          $i = az group exists -n ${{ parameters.RGName }}
          if ($i -eq "true") { 
          $j = az container list --query "[?contains(name, '${{ parameters.ACIName }}')].[provisioningState]" -o tsv
            if ($j -eq "Succeeded") {
            echo "##########################################################################################################################################"
            echo "${{ parameters.ACIName }} CONTAINER INSTANCE EXISTS IN THE ${{ parameters.RGName }} RESOURCE GROUP. CANNOT PROCEED WITH THE DEPLOYMENT"
            echo "##########################################################################################################################################"  
              }
            else {   
            az container create -g ${{ parameters.RGName }} --name ${{ parameters.ACIName }} --image sonarqube --ports $(port) --dns-name-label ${{ parameters.ACIName }} --cpu $(cpu) --memory $(memory)
              }
            }             
          else {
          az group create -l ${{ parameters.RGLocation }} -n ${{ parameters.RGName }} 
          az container create -g ${{ parameters.RGName }} --name ${{ parameters.ACIName }} --image sonarqube --ports $(port) --dns-name-label ${{ parameters.ACIName }} --cpu $(cpu) --memory $(memory)    
            }

```

Let me explain each part of YAML Pipeline for better understanding.

| __PART #1:-__|
| --------- |

| BELOW FOLLOWS PIPELINE RUNTIME VARIABLES CODE SNIPPET:- |
| --------- |

```
######################
#DECLARE PARAMETERS:-
######################
parameters:
- name: SubscriptionID
  displayName: Please Provide the Subscription ID:-
  type: object
  default: 210e66cb-55cf-424e-8daa-6cad804ab604

- name: ServiceConnection
  displayName: Please Provide the Service Connection Name:-
  default: amcloud-cicd-service-connection
  values:
  - amcloud-cicd-service-connection

- name: RGName
  displayName: Please Provide the Name of the Resource Group:-
  type: object
  default: DemoRGSonarQube

- name: RGLocation
  displayName: Please Provide Location of Resource Group:-
  default: WestEurope
  values:
  - WestEurope

- name: validateIfRGExists
  displayName: Does the provided Resource Group Exists or do we create ?
  default: Existing
  values:
  - Existing
  - New

- name: ACIName
  displayName: Please Provide the Name of the Azure Container Instance:-
  type: object
  default: am-poc-sonarqube-aci
```

| THIS IS HOW IT LOOKS WHEN YOU EXECUTE THE PIPELINE FROM AZURE DEVOPS:- |
| --------- |

| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4pi7izqn32yxszwzimsd.png) |
| --------- |

| NOTE:- |
| --------- |


| Please feel free to change the name of the __RESOURCE GROUP NAME__ and the __NAME OF THE AZURE CONTAINER INSTANCE.__ |
| --------- |

| __PART #2:-__|
| --------- |

| BELOW FOLLOWS PIPELINE VARIABLES CODE SNIPPET:- |
| --------- |

```
######################
#DECLARE VARIABLES:-
######################
variables:
  Artifact: AM
  cpu: 2 
  memory: 3.5
  port: 9000

```

| NOTE:- |
| --------- |

| Please feel free to change the values of the variables. |
| --------- |
| __The entire YAML pipeline is build using Parameters and variables. No Values are Hardcoded.__ |


| __PART #3:-__|
| --------- |

| PIPELINE STAGES:- |
| --------- |
| There are 2 Stages in the Pipeline 1) When User selects "__EXISTING__" Pipeline Runtime Environment 2) When User selects "__NEW__" Pipeline Runtime Environment|
| Pipeline Stage gets __SKIPPED__ or __EXECUTED__ based on the User Choice  |

| BELOW FOLLOWS PIPELINE STAGE __"EXISTING"__ CODE SNIPPET:- |
| --------- |

```
- stage: VALIDATE_SELECTION_EXISTING
  condition: |
     and(eq('${{ parameters.validateIfRGExists }}', 'Existing'), 
       eq(variables['build.sourceBranch'], 'refs/heads/main')) 
  jobs:
  - job: IF_RG_EXISTS_DEPLOY_SONARQUBE 
    displayName: IF RG EXISTS DEPLOY SONARQUBE
    steps:
    - task: AzureCLI@2
      displayName: CHECK RG AND DEPLOY SONARQUBE
      inputs:
        azureSubscription: ${{ parameters.ServiceConnection }}
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show  
          $i = az group exists -n ${{ parameters.RGName }}
          if ($i -eq "true") {
          $j = az container list --query "[?contains(name, '${{ parameters.ACIName }}')].[provisioningState]" -o tsv
            if ($j -eq "Succeeded") {
            echo "##########################################################################################################################################"
            echo "${{ parameters.ACIName }} CONTAINER INSTANCE EXISTS IN THE ${{ parameters.RGName }} RESOURCE GROUP. CANNOT PROCEED WITH THE DEPLOYMENT"
            echo "##########################################################################################################################################"  
              }
            else {   
            az container create -g ${{ parameters.RGName }} --name ${{ parameters.ACIName }} --image sonarqube --ports $(port) --dns-name-label ${{ parameters.ACIName }} --cpu $(cpu) --memory $(memory)
              }
            }
          else {
          az group create -l ${{ parameters.RGLocation }} -n ${{ parameters.RGName }} 
          az container create -g ${{ parameters.RGName }} --name ${{ parameters.ACIName }} --image sonarqube --ports $(port) --dns-name-label ${{ parameters.ACIName }} --cpu $(cpu) --memory $(memory)    
            }

```

| BELOW FOLLOWS PIPELINE STAGE __"NEW"__ CODE SNIPPET:- |
| --------- |

```
- stage: VALIDATE_SELECTION_NEW
  condition: |
     and(eq('${{ parameters.validateIfRGExists }}', 'New'), 
       eq(variables['build.sourceBranch'], 'refs/heads/main')) 
  jobs:
  - job: CREATE_RG_AND_DEPLOY_SONARQUBE 
    displayName: CREATE RG AND DEPLOY SONARQUBE
    steps:
    - task: AzureCLI@2
      displayName: CHECK RG AND DEPLOY SONARQUBE
      inputs:
        azureSubscription: ${{ parameters.ServiceConnection }}
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az --version
          az account set --subscription ${{ parameters.SubscriptionID }}
          az account show  
          $i = az group exists -n ${{ parameters.RGName }}
          if ($i -eq "true") { 
          $j = az container list --query "[?contains(name, '${{ parameters.ACIName }}')].[provisioningState]" -o tsv
            if ($j -eq "Succeeded") {
            echo "##########################################################################################################################################"
            echo "${{ parameters.ACIName }} CONTAINER INSTANCE EXISTS IN THE ${{ parameters.RGName }} RESOURCE GROUP. CANNOT PROCEED WITH THE DEPLOYMENT"
            echo "##########################################################################################################################################"  
              }
            else {   
            az container create -g ${{ parameters.RGName }} --name ${{ parameters.ACIName }} --image sonarqube --ports $(port) --dns-name-label ${{ parameters.ACIName }} --cpu $(cpu) --memory $(memory)
              }
            }             
          else {
          az group create -l ${{ parameters.RGLocation }} -n ${{ parameters.RGName }} 
          az container create -g ${{ parameters.RGName }} --name ${{ parameters.ACIName }} --image sonarqube --ports $(port) --dns-name-label ${{ parameters.ACIName }} --cpu $(cpu) --memory $(memory)    
            }

```

| __PART #4:-__|
| --------- |

| PIPELINE STAGE __VALIDATE_SELECTION_EXISTING__ CONDITIONS:-|
| --------- |

| __##__ | __CONDITIONS APPLIED__ | 
| --------- | --------- |
| 1. | When User selects __"Existing"__ Pipeline Runtime Environment, it means that the mentioned RESOURCE GROUP __(RG)__ Exists in the Subscription.  |
| 2. | Pipeline when executed first validates if __RG__ Exists.  |
| 3. | If __RG EXISTS__, it will then validate if the mentioned Azure Container Instance __(ACI)__, provided using Pipeline Runtime Variables exists. If __ACI__ exists, No Action to be Taken. If __ACI__ does not Exists, Deploy __ACI with SonarQube Image__ |
| 4. | If __RG DOES NOT EXISTS__, it will first deploy __RG__ and then __ACI with SonarQube Image__ |


| __PART #5:-__|
| --------- |

| PIPELINE STAGE __VALIDATE_SELECTION_NEW__ CONDITIONS:-|
| --------- |

| __##__ | __CONDITIONS APPLIED__ | 
| --------- | --------- |
| 1. | When User selects __"New"__ Pipeline Runtime Environment, it means that the mentioned RESOURCE GROUP __(RG)__ does not Exists in the Subscription.  |
| 2. | Pipeline when executed first validates if __RG__ Exists.  |
| 3. | If __RG EXISTS__, it will then validate if the mentioned Azure Container Instance __(ACI)__, provided using Pipeline Runtime Variables exists. If __ACI__ exists, No Action to be Taken. If __ACI__ does not Exists, Deploy __ACI with SonarQube Image__ |
| 4. | If __RG DOES NOT EXISTS__, it will first deploy __RG__ and then __ACI with SonarQube Image__ |


| __PART #6:-__|
| --------- |

__VALIDATE THE YAML DEPLOYMENT WITH BELOW TEST CASES.__


| __TEST CASE #1:__ SELECT "NEW" (There is No RG and ACI with SonarQube Image):-|
| --------- |
| __Desired Output:__ RG + ACI with SonarQube Image Gets Deployed. |
| __PIPELINE VARIABLES:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wb8wjewchdk1ul1og5dk.png) |
| __PIPELINE RESULTS:__|
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kqqwoy8fqy4bg5pyrcih.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/srkt1d3mstcvhbehqzdf.png) |
| __SonarQube URL:__ http://am-poc-sonarqube-aci.westeurope.azurecontainer.io:9000/ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/girui3gac12mevw6v60l.png) |
 

| __TEST CASE #2:__ SELECT "EXISTING" (RG and ACI with SonarQube Image already Exists):-|
| --------- |
| __Desired Output:__ No Action Performed. |
| __PIPELINE VARIABLES:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w0gw0ahz27ck6dh63rbh.png) |
| __PIPELINE RESULTS:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k0919c40dnvyuy8smsgc.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ryvosqa56uf70jt5btyx.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/awi8y0xffe11x81cuyug.png) |
| __SonarQube URL:__ http://am-poc-sonarqube-aci.westeurope.azurecontainer.io:9000/ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/girui3gac12mevw6v60l.png) |

| __TEST CASE #3:__ SELECT "NEW" (RG and ACI with SonarQube Image already Exists):-|
| --------- |
| __Desired Output:__ No Action Performed. |
| __PIPELINE VARIABLES:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wb8wjewchdk1ul1og5dk.png) |
| __PIPELINE RESULTS:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kqqwoy8fqy4bg5pyrcih.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/srkt1d3mstcvhbehqzdf.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q2mkuvc42vwgxusrsk1r.png) |
| __SonarQube URL:__ http://am-poc-sonarqube-aci.westeurope.azurecontainer.io:9000/ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/girui3gac12mevw6v60l.png) |

| __TEST CASE #4:__ SELECT "EXISTING" (RG Exists but there is No ACI with SonarQube Image):-|
| --------- |
| __Desired Output:__ ACI with SonarQube Image gets Deployed Only. |
| __PIPELINE VARIABLES:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w0gw0ahz27ck6dh63rbh.png) |
| __PIPELINE RESULTS:__ |
|![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k0919c40dnvyuy8smsgc.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ryvosqa56uf70jt5btyx.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/otw1ymq1vy208uoyb4um.png) |
| __SonarQube URL:__ http://am-poc-sonarqube-aci.westeurope.azurecontainer.io:9000/ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/girui3gac12mevw6v60l.png) |


| __TEST CASE #5:__ SELECT "NEW" (RG Exists but there is No ACI with SonarQube Image):-|
| --------- |
| __Desired Output:__ ACI with SonarQube Image gets Deployed Only. |
| __PIPELINE VARIABLES:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wb8wjewchdk1ul1og5dk.png) |
| __PIPELINE RESULTS:__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kqqwoy8fqy4bg5pyrcih.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/srkt1d3mstcvhbehqzdf.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/otw1ymq1vy208uoyb4um.png) |
| __SonarQube URL:__ http://am-poc-sonarqube-aci.westeurope.azurecontainer.io:9000/ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/girui3gac12mevw6v60l.png) |


Hope You Enjoyed the Session!!!

__Stay Safe | Keep Learning | Spread Knowledge__
