# Usage2ADWFunc - Oracle Cloud Infrastructure Usage and Cost Reports to Autonomous Database with APEX Reporting

## Step by Step Manual installation Guide on OCI Function and Autonomous Data Warehouse Database
usage2adwfnuc is a tool which uses the Python SDK to extract the usage reports from your tenant and load it to Oracle Autonomous Database.

Oracle Application Express (APEX) will be used for reporting.  

**DISCLAIMER – This is not an official Oracle application,  It does not supported by Oracle Support, It should NOT be used for utilization calculation purposes, and rather OCI's official 
[cost analysis](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/costanalysisoverview.htm) 
and [usage reports](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/usagereportsoverview.htm) features should be used instead.**

**Developed by Scott herman, 2023**  Bassed on [usage_to_adw](https://github.com/oracle/oci-python-sdk/tree/master/examples/usage_reports_to_adw) python example.

## 1. Deploy Function to run the python script

```
OCI -> Menu -> Developer Services -> Functions -> Applications
Create applicaiton
--> Name = Usage2ADW
--> Choose your network VCN and Subnet (any type of VCN and Subnet)
--> Press Create

Copy Applicaiton Info:
--> Applicaiton OCID to be used for Dynamic Group Permission

```

## 2. Create Secret for Storing Database Password

```
OCI -> Menu -> Identity & Security -> Vault
Create Vault
--> Name = DatabaseUsers

Create Key
--> Name = MasterKey

-> Secrets
Create Secret
--> Name = usage_user
--> Dewscription = Password for database user that holds Usage2ADWFunc
--> Encryption Key = MasterKey
--> Secret Contents = {USAGE_PASSWORD}

Copy Secret Info:
--> Secret OCID to be used for Function Configuration
```

## 3. Create Dynamic Group for Instance Principles

```
OCI -> Menu -> Identity -> Dynamic Groups -> Create Dynamic Group
--> Name = UsageDownloadGroup 
--> Desc = Dynamic Group for the Usage Report VM
--> Rule 1 = ALL {resource.id = 'OCID_Of_Step_1_Function'}
```

## 4. Create Policy to allow the Dynamic Group to extract usage report and read Compartments

```
OCI -> Menu -> Identity -> Policies
Choose Root Compartment
Create Policy
--> Name = UsageDownloadPolicy
--> Desc = Allow Dynamic Group UsageDownloadGroup to Extract Usage report script
--> Statement 1 = define tenancy usage-report as ocid1.tenancy.oc1..aaaaaaaaned4fkpkisbwjlr56u7cj63lf3wffbilvqknstgtvzub7vhqkggq
--> Statement 2 = endorse dynamic-group UsageDownloadGroup to read objects in tenancy usage-report
--> Statement 3 = Allow dynamic-group UsageDownloadGroup to inspect compartments in tenancy
--> Staement 4 = Allow dynamic-group UsageDownloadGroup to inspect tenancies in tenancy
--> Statement 5 = Allow dynamic-group UsageDownloadGroup to read autonomous-databases in compartment {APPCOMP}
--> Statement 6 = Allow dynamic-dynamic-group UsageDownloadGroup to read secret-family in compartment {VAULTCOMP} where target.secret.name = 'usage_user'
*** Please don't change the usage report tenant OCID, it is fixed.
```

## 5. Deploy Autonomous Data Warehouse Database

```
OCI -> Menu -> Autonomous Data Warehouse
Create Autonomous Database
--> Compartment = Please Choose
--> Display Name = ADWCUSG
--> Database Name ADWCUSG
--> Workload = Data Warehouse
--> Deployment = Shared
--> Always Free = Optional
--> OCPU = 1
--> Storage = 1
--> Auto Scale = No
--> Password = (Please choose your own password)
--> Choose Network Access = Allow secure Access from Everywhere (you can use VCN as well which requires NSG)
--> Choose License Type
```

## 5. Setup fn CLI on Cloud Shell

```
In Cloud Shell run these commands for setting up environment for deploying function.  This information is on the getting started section for Applicaion Seciton in OCI Steps 1 - 7
```

## 6. Clone the OCI Usage Funcrtion Repo from Git Hub
  
```
cd $HOME
git clone https://github.com/hermansd/oci-usage-function.git
cd oci-usage-function/Uasage2ADWFunc
```

## 7. Deploy Usage2ADW Function
```
# Replace TENANCY_OCID, ADW_OCID and SECRET_OCID wih values from your environment

fn -v deploy --app Usage2ADW
fn config function Usage2ADW adw-billing usage_report_bucket {TENANCY_OCID}
fn config function Usage2ADW adw-billing TNS_ADMIN /tmp/wallet
fn config function Usage2ADW adw-billing db_dsn psadwwest_high
fn config function Usage2ADW adw-billing db_ocid {ADW_OCID}
fn config function Usage2ADW adw-billing db_user USAGE
fn config function Usage2ADW adw-billing skip_usage  False
fn config function Usage2ADW adw-billing skip_cost  False
fn config function Usage2ADW adw-billing skip_rate  False
fn config function Usage2ADW adw-billing force False
fn config function Usage2ADW adw-billing secret_ocid {SECRET_OCID}
```

## 8. Create Database User for the Usage repository

```
OCI -> Menu -> Autonomous Data Warehouse -> ADWCUSG -> Database Actions -> SQL

# Choose your own password and run
create user usage identified by <password>;
grant connect, resource, dwrole, unlimited tablespace to usage;
```


## 9. Invoke Usage2ADW Function
```
# Open Cloud Shell
fn invoke Usage2ADW adw-billing
```

## Todo:
- [ ] APEX application deployment
- [ ] APEX Scheduling of function


## License

Copyright (c) 2016, 2023, Oracle and/or its affiliates.  All rights reserved.
This software is dual-licensed to you under the Universal Permissive License (UPL) 1.0 as shown at https://oss.oracle.com/licenses/upl
or Apache License 2.0 as shown at http://www.apache.org/licenses/LICENSE-2.0. You may choose either license.