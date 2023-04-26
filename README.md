# PeopleSoft Enterprise Monitoring
Oracle Logging Analytics can help you identify PeopleSoft Enterprise issues to make your business operations smooth to delight your customers. This resource manager stack configures logging analytics to enable these functional sensors such as:

Vouchers stuck in processing (PeopleSoft Financials Payables)
Financial Aid Term Mismatch (PeopleSoft Campus Solutions Financial Aid)
Formulas not validated (PeopleSoft HCM Global Payroll)
IB transactions stuck (PeopleSoft PeopleTools)
And 10s of other issues.
This OCI Resource Manager stack creates an instance in the subnet from which PSFT Database is accessible, configures Management Agent and Logging Analytics to run regular Business checks. It installs four new dashboards covering different tiers of PSFT deployment. Note: This stack doesn't start standard Logs collection and Logging Analytics PSFT Discovery from the UI should be used to discover and enable logs collection.

As part of this deployment, a compute instance is created and Oracle Cloud Agent is configured to collect log data. Users can select the PSFT products that they are using and PSFT sensor sources for those products are created. PSFT Database entity and source-entity associations are also created.  

## Prerequisites
- VCN and subnet from where database can be accessed.
- The subnet should have access to OCI Services (via a Service Gateway)
- Quota to create the following resources: 1 Compute instance,  1 dynamic group, 1 policy
- Store PSFT DB password in OCI Vault in base encoded form. 
- Store schedule file in OCI Object Storage bucket. 

If you don't have the required permissions and quota, contact your tenancy administrator. See [Policy Reference](https://docs.cloud.oracle.com/en-us/iaas/Content/Identity/Reference/policyreference.htm), [Service Limits](https://docs.cloud.oracle.com/en-us/iaas/Content/General/Concepts/servicelimits.htm), [Compartment Quotas](https://docs.cloud.oracle.com/iaas/Content/General/Concepts/resourcequotas.htm).

## Deploy Using OCI Resource Manager

1. Click [![Deploy to Oracle Cloud](https://oci-resourcemanager-plugin.plugins.oci.oraclecloud.com/latest/deploy-to-oracle-cloud.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/oracle-quickstart/oci-ebs-monitoring/releases/download/v0.9/ebs-v0.9.zip)

#    If you aren't already signed in, when prompted, enter the tenancy and user credentials.

2. Review and accept the terms and conditions.

3. Select the region where you want to deploy the stack.

4. Follow the on-screen prompts and instructions to create the stack.

5. After creating the stack, click **Terraform Actions**, and select **Plan**.

6. Wait for the job to be completed, and review the plan.

    To make any changes, return to the Stack Details page, click **Edit Stack**, and make the required changes. Then, run the **Plan** action again.

7. If no further changes are necessary, return to the Stack Details page, click **Terraform Actions**, and select **Apply**.

## Deploy Using the Terraform CLI

### Clone the Module
Now, you'll want a local copy of this repo. You can make that with the commands:

    git clonehttps://github.com/oracle-quickstart/oci-psft-monitoring.git
    cd oci-psft-monitoring
    ls
  
### Set Up and Configure Terraform

1. Complete the prerequisites described [here](https://github.com/cloud-partners/oci-prerequisites).

2. Create a `terraform.tfvars` file, and specify the following variables:

```
# Authentication
tenancy_ocid="<tenancy_ocid>"
auth_type="user"
# Config  file is ~/.oci/config 
config_file_profile="DEFAULT"

# Region
region = "<oci_region>"

# Compartment
compartment_ocid = "<compartment_ocid>"

# PSFT DB Info
subnet_ocid="<DB_NETWORK_SUBNET_OCID>"
db_compartment="<DB_COMPARTMENT_OCID>"
db_cred_compartment="<VAULT_COMPARTMENT_OCID>"

db_host="<DB_HOST>"
db_port=<DB_PORT>
db_service="<DB_SERVICE_NAME>"
db_username="<DB_USER_NAME>"
db_credentials="<VAULT_SECRET_OCID>"
db_user_role="NORMAL"

#Agent Compute Instance Info
instance_name="PSFTAgentVM"
availability_domain="<AD>"
instance_shape="VM.Standard.E2.1"
user_ssh_secret="<SSH-KEY>"

# Set to false if you want to manually create dynamic group and policies
setup_policies=true

# Location of schedule file
bucket_name="<BUCKET_NAME>"
file_name="logan_schedule_database_sql_PSFT.csv"

# Log Analytics Resources
resource_compartment="<RESOURCE_COMPARTMENT_OCID>"
create_log_group=false
log_group_ocid="<LOG_GROUP_OCID>"
la_entity_name = "<PSFTDB entity name>"

# Selected products
#products="PeopleSoft Enterprise PT PeopleTools"

products="PeopleSoft Enterprise CS Financial Aid,PeopleSoft Enterprise CS Student Financials,PeopleSoft Enterprise CS Student Records,PeopleSoft Enterprise FIN Asset Management,PeopleSoft Enterprise FIN Contracts,PeopleSoft Enterprise FIN Expenses,PeopleSoft Enterprise FIN General Ledger,PeopleSoft Enterprise FIN Payables,PeopleSoft Enterprise FIN Project Costing,PeopleSoft Enterprise HCM Benefits,PeopleSoft Enterprise HCM Benefits Administration,PeopleSoft Enterprise HCM Global Payroll Core,PeopleSoft Enterprise HCM Human Resources,PeopleSoft Enterprise HCM Payroll for North America,PeopleSoft Enterprise HCM Time and Labor,PeopleSoft Enterprise PT PeopleTools,PeopleSoft Enterprise SCM Purchasing"

### Create the Resources
Run the following commands:

    terraform init
    terraform plan
    terraform apply


```
Outputs:


### Destroy the Deployment
When you no longer need the deployment, you can run this command to destroy the resources:

    terraform destroy

### Dynamic Groups and Policies (if adding manually)

1. Create a dynamic group instance dynamic group with matching rule:
- ANY {instance.compartment.id = '<db_compartment_ocid>'}
2. Create dynamic group mgmtagent dynamic group with matching rule:
- ALL {resource.type='managementagent', resource.compartment.id='<db_compartment_ocid>'}
3. Create a policy at tenancy level with the following statements:
- Allow DYNAMIC-GROUP <mgmtagent_dynamic_group_name> to {LOG_ANALYTICS_LOG_GROUP_UPLOAD_LOGS} in tenancy
- ALLOW DYNAMIC-GROUP <mgmtagent_dynamic_group_name> TO MANAGE management-agents IN COMPARTMENT ID <db_compartment_ocid>
- ALLOW DYNAMIC-GROUP <mgmtagent_dynamic_group_name> TO USE METRICS IN COMPARTMENT ID <db_compartment_ocid>
- ALLOW DYNAMIC-GROUP <instance_dynamic_group_name> TO MANAGE management-agents IN COMPARTMENT ID <db_compartment_ocid>
- ALLOW DYNAMIC-GROUP <instance_dynamic_group_name> TO MANAGE management-agent-install-keys IN COMPARTMENT ID <db_compartment_ocid>
- ALLOW DYNAMIC-GROUP <instance_dynamic_group_name> TO MANAGE OBJECTS IN COMPARTMENT ID <db_compartment_ocid>
- ALLOW DYNAMIC-GROUP <instance_dynamic_group_name> TO READ BUCKETS IN COMPARTMENT ID <db_compartment_ocid>
- ALLOW DYNAMIC-GROUP <instance_dynamic_group_name> TO READ secret-family in COMPARTMENT ID <vault_compartment_ocid>} where target.secret.id = '<db_secret_ocid>'


