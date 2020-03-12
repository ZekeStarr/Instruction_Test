###Redeem Your Azure Pass

1. [] In the open browser window, click the **Start** button

    !IMAGE[irt1zvvs.jpg](irt1zvvs.jpg)

1. [] Enter the credential information provided and click **Sign In**

    !IMAGE[3q4jd78h.jpg](3q4jd78h.jpg)

    - **Username**: ++++@lab.CloudCredential(Ready2020SeattleM365EnterpriseBlankAdminOnly).Username++++
    - **Password**: ++++@lab.CloudCredential(Ready2020SeattleM365EnterpriseBlankAdminOnly).Password++++ 

    >[!alert] If you have stored credentials, click **Sign In as different user** before signing in.

1. [] Click Confirm Microsoft Account if the email matches @lab.CloudCredential(Ready2020SeattleM365EnterpriseBlankAdminOnly).Username

1. [] Copy ++@lab.CloudCredential(Ready2020SeattleAzurePasses).PromoCode++ and paste it into the promo code box. Then click **Claim Promo Code**

    !IMAGE[yql1vqv7.jpg](yql1vqv7.jpg)

    >[!alert] It can take 5 mins or more for the subscription to provision. Please be patient and do not navigate away from this page.
    >
    >!IMAGE[ou3r615d.jpg](ou3r615d.jpg)
    >
    >You will automaticvally be redirected to a Sign Up page when the process is completed.

1. [] Once you are redirected to the Sign Up page, fill out the "About You" section and press **Next**

1. [] Check the agreement box and click **Sign Up**.

1. [] Once you are at the Azure Portal page, you are ready to continue.

=== 

##**Deploy a single node HANA**

---


##**Exercise 1: Deploy a single node HANA instance by using Terraform and Ansible**

 

Duration: 50 minutes

 

In this exercise, you will implement a single-node deployment of SAP HANA on Azure virtual machines (VMs). Following initial configuration of Terraform-based templates, the deployment will be fully automated, including installation of all necessary SAP HANA components.

**Task 1 : Prepare for a single node HANA deployment**

1. [] From the lab computer, start a Web browser, and navigate to the Azure portal at ++++https://portal.azure.com++++.

1. [] In the Azure portal at *http://portal.azure.com*, click the Cloud Shell icon.

    !IMAGE[image2.png](image2.png)

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to generate the SSH key pair that will be used to secure access to Linux Azure VMs deployed in this lab:

    > ++++ssh-keygen -t rsa -b 2048++++

1. [] When prompted to specify the location of the file in which to save the key and to specify the passphrase protecting the content of the file, press the Enter key (three times). You should see the output similar to the following:

    > Generating public/private rsa key pair.  
    > Enter file in which to save the key (/home/m/.ssh/id_rsa):  
    > Created directory '/home/m/.ssh'.  
    > Enter passphrase (empty for no passphrase):  
    > Enter same passphrase again:  
    > Your identification has been saved in /home/m/.ssh/id_rsa.  
    > Your public key has been saved in /home/m/.ssh/id_rsa.pub.  
    > The key fingerprint is:  
    > SHA256:2gAFQbAc2QFR4miR49Hdk6zyPH7YLWvbjiINEH5WScA m@cc-87c3f8bb-f6549dcd6-8mf42  
    > The key's randomart image is:  
    > +---[RSA 2048]----+  
    > | .XXX== .        |  
    > | OoE.= =         |  
    > |+.B o . .        |  
    > |.+ + o           |  
    > |  + + . S        |  
    > |   . + +         |  
    > |    + = o        |  
    > |   . = =o.       |  
    > |    . ++=o       |  
    > +----[SHA256]-----+

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to create an Azure AD service principal that will be used during deployment:

    ```
    HANA_SP_NAME='sp-hanaholv1'  
    HANA_SP_ID=$(az ad sp list --display-name $HANA_SP_NAME --query "[0].appId" --output tsv)  
    if ! [ -z "$HANA_SP_ID"]  
    then  
        az ad sp delete --id $HANA_SP_ID  
    fi  
    HANA_SP=$(az ad sp create-for-rbac --name $HANA_SP_NAME)
    ```

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to set the variables that that will be used during deployment, representing, respectively, the identifier of the Azure subscription and its Azure AD tenant, as well as the application identifier and the corresponding password of the service principal you created in the previous step:

    ```
    export AZURE_SUBSCRIPTION_ID=$(az account show | jq -r '.id')  
    export AZURE_TENANT=$(az account show | jq -r '.tenantId')  
    export AZURE_CLIENT_ID=$(echo $HANA_SP | jq -r '.appId')  
    export AZURE_SECRET=$(echo $HANA_SP | jq -r '.password')  
       
    export ARM_SUBSCRIPTION_ID=$(az account show | jq -r '.id')  
    export ARM_TENANT_ID=$(az account show | jq -r '.tenantId')  
    export ARM_CLIENT_ID=$(echo $HANA_SP | jq -r '.appId')  
    export ARM_CLIENT_SECRET=$(echo $HANA_SP | jq -r '.password')
    ```
1. [] In the Cloud Shell pane, from the Bash prompt, run the following to clone the repository hosting the Terraform and Ansible files that you will use for deployment:  
    
    ++++rm ~/sap-hana/ -r -f++++

    ++++git clone https://github.com/Azure/sap-hana.git++++
    
1. [] In the Cloud Shell pane, from the Bash prompt, run the following to change the current directory to the one hosting the Terraform and Ansible files that you will use for deployment:  
      
    > ++++cd sap-hana/deploy/vm/modules/single_node_hana/++++

1. [] In the Cloud Shell menu bar, upload file "terraform.tfvars" (attached below) to the home directory of the Bash session in the Cloud Shell. 

    <[terraform.tfvars](https://labondemand.blob.core.windows.net/content/lab62680/resources/terraform.tfvars)

    
    !image[image3.jpeg](image3.jpeg)
      
    
    !image[image4.jpeg](image4.jpeg)
  

    Then in the Cloud Shell pane, from the Bash prompt, copy terraform.tfvars file to the directory when you run Terraform for deployment.

    >++++cp ~/terraform.tfvars ~/sap-hana/deploy/vm/modules/single_node_hana++++

 

1. [] Again in the Cloud Shell menu bar, upload file "nsg.tf" (attached below) to the home directory of the Bash session in the Cloud Shell.  

    <[nsg.tf](https://labondemand.blob.core.windows.net/content/lab62680/resources/nsg.tf)

    Then in the Cloud Shell pane, from the Bash prompt, copy nsg.tf file to the directory when you run Terraform for deployment.

    ++++cp ~/nsg.tf  ~/sap-hana/deploy/vm/modules/common_setup++++

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to specify the location in which you created the storage account in the previous task:  
    
    > ++++LOCATION='southcentralus'++++

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to create the resource group that will host all resources deployed in this task:  

    ```
    HANA_V1_SN_RESOURCE_GROUP_NAME='rsg-hanaholv1'  
    az group create --location $LOCATION --name $HANA_V1_SN_RESOURCE_GROUP_NAME  
    ```
    
    >[!KNOWLEDGE] If needed, this step can be automated as well. Terraform (as well as Azure Resource Manager) templates can be configured to create resource groups.

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to generate a pseudo-random name that will be used as a prefix for DNS names assigned to public IP address resources deployed in this task:  
       
    >++++DOMAIN_NAME=domhanaholv1$RANDOM++++

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to specify the size of the Azure VM to be used to host the single node HANA instance deployed in this task:  
       
    >++++VM_SIZE='Standard_E8s_v3'++++

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to specify the name of the root user account for the Linux VM deployed in this task:  
       
    >++++VM_USERNAME='labuser'++++

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to add the values you specified to the terraform.tfvars file that contains variables used by Terraform deployment performed in this task:

    ```
    sed -i "s/VAR_RESOURCE_GROUP/$HANA_V1_SN_RESOURCE_GROUP_NAME/" ./terraform.tfvars  
    sed -i "s/VAR_LOCATION/$LOCATION/" ./terraform.tfvars  
    sed -i "s/VAR_DOMAIN_NAME/$DOMAIN_NAME/" ./terraform.tfvars  
    sed -i "s/VAR_VM_SIZE/$VM_SIZE/" ./terraform.tfvars  
    sed -i "s/VAR_VM_USERNAME/$VM_USERNAME/" ./terraform.tfvars
    ```
1. [] In the Cloud Shell pane, from the Bash prompt, run the following to add the values you specified to the terraform.tfvars file that contains variables used by Terraform deployment performed in this task:  

    ```
    SAPCAR_LINUX_URL='https://strholterraformsc.blob.core.windows.net/sapbits/SAPCAR_1311-80000935.EXE?sp=r&st=2020-01-11T19:17:05Z&se=2020-07-01T02:17:05Z&spr=https&sv=2019-02-02&sr=b&sig=A%2FETzsb%2FDDs0W5ePe6QsmTSNJ4d%2BwIMJvrzqbRPQiO0%3D'
    SAPCAR_LINUX_URL_REGEX="$(echo $SAPCAR_LINUX_URL | sed -e 's/\\/\\\\/g; s/\//\\\//g; s/&/\\\&/g')"
    SAPCAR_WINDOWS_URL='https://strholterraformsc.blob.core.windows.net/sapbits/SAPCAR_1311-80000938.EXE?sp=r&st=2020-01-11T19:18:58Z&se=2020-07-01T02:18:58Z&spr=https&sv=2019-02-02&sr=b&sig=oMMi216dxMAgl%2B1szeY9D5AaOjOLZOEIDkNKFG1E3q0%3D'
    SAPCAR_WINDOWS_URL_REGEX="$(echo $SAPCAR_WINDOWS_URL | sed -e 's/\\/\\\\/g; s/\//\\\//g; s/&/\\\&/g')"

    HDBSERVER_URL='https://strholterraformsc.blob.core.windows.net/sapbits/IMDB_SERVER100_122_29-10009569.SAR?sp=r&st=2020-01-11T19:20:01Z&se=2020-07-01T02:20:01Z&spr=https&sv=2019-02-02&sr=b&sig=cB7%2BRUBoW5nbJ6aEy3LJYA9se9KUL4J52YCL5dw1eJs%3D'
    HDBSERVER_URL_REGEX="$(echo $HDBSERVER_URL | sed -e 's/\\/\\\\/g; s/\//\\\//g; s/&/\\\&/g')"

    SAP_HOST_AGENT_URL='https://strholterraformsc.blob.core.windows.net/sapbits/SAPHOSTAGENT36_36-20009394.SAR?sp=r&st=2020-01-11T19:20:52Z&se=2020-07-01T02:20:52Z&spr=https&sv=2019-02-02&sr=b&sig=lODtKMvAmqSsbDQ65RqIbtv%2F46o4qjn6B65N9RTDQNA%3D'
    SAP_HOST_AGENT_URL_REGEX="$(echo $SAP_HOST_AGENT_URL | sed -e 's/\\/\\\\/g; s/\//\\\//g; s/&/\\\&/g')"

    HANA_STUDIO_WINDOWS_URL='https://strholterraformsc.blob.core.windows.net/sapbits/IMC_STUDIO2_245_0-80000323.SAR?sp=r&st=2020-01-11T19:21:34Z&se=2020-07-01T02:21:34Z&spr=https&sv=2019-02-02&sr=b&sig=zDZFJFLdjCNcHJrITPk3IEAw17v9B1S1pEhBxvJmCCA%3D'
    HANA_STUDIO_WINDOWS_URL_REGEX="$(echo $HANA_STUDIO_WINDOWS_URL | sed -e 's/\\/\\\\/g; s/\//\\\//g; s/&/\\\&/g')"

    sed -i "s/VAR_SAPCAR_LINUX_URL/$SAPCAR_LINUX_URL_REGEX/" ./terraform.tfvars
    sed -i "s/VAR_SAPCAR_WINDOWS_URL/$SAPCAR_WINDOWS_URL_REGEX/" ./terraform.tfvars
    sed -i "s/VAR_HDBSERVER_URL/$HDBSERVER_URL_REGEX/" ./terraform.tfvars
    sed -i "s/VAR_SAP_HOST_AGENT_URL/$SAP_HOST_AGENT_URL_REGEX/" ./terraform.tfvars
    sed -i "s/VAR_HANA_STUDIO_WINDOWS_URL/$HANA_STUDIO_WINDOWS_URL_REGEX/" ./terraform.tfvars

    ```

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to set the values of passwords for user accounts that will be used to manage the single-node HANA instance:  
    ```
    SAPADMUSER_PASSWORD='Demo@pass123'
    ADMUSER_PASSWORD='Demo@pass123'
    DBSYSTEMUSER_PASSWORD='Demo@pass123'
    DBXSAADMIN_PASSWORD='Demo@pass123'
    DBSYSTEMTENANTUSER_PASSWORD='Demo@pass123'
    DBSHINEUSER_PASSWORD='Demo@pass123'
    WINDOWS_ADMIN_PASSWORD='Demo@pass123'
    ```

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to add the password values to the terraform.tfvars file that contains variables used by Terraform deployment performed in this task:  
    
    ```
    sed -i "s/VAR_SAPADMUSER_PASSWORD/$SAPADMUSER_PASSWORD/" ./terraform.tfvars  
    sed -i "s/VAR_ADMUSER_PASSWORD/$ADMUSER_PASSWORD/" ./terraform.tfvars  
    sed -i "s/VAR_DBSYSTEMUSER_PASSWORD/$DBSYSTEMUSER_PASSWORD/" ./terraform.tfvars  
    sed -i "s/VAR_DBXSAADMIN_PASSWORD/$DBXSAADMIN_PASSWORD/" ./terraform.tfvars  
    sed -i "s/VAR_DBSYSTEMTENANTUSER_PASSWORD/$DBSYSTEMTENANTUSER_PASSWORD/" ./terraform.tfvars  
    sed -i "s/VAR_DBSHINEUSER_PASSWORD/$DBSYSTEMTENANTUSER_PASSWORD/" ./terraform.tfvars  
    sed -i "s/VAR_WINDOWS_ADMIN_PASSWORD/$WINDOWS_ADMIN_PASSWORD/" ./terraform.tfvars
    ```

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to check terraform.tfvars file. Make sure az_region, az_resource_group, az_domain_name, vm_size, vm_user, all URL parameters, all password parameters are correctly updated.  

    > ++++cat terraform.tfvars++++

     The file should look like this. 

    <[terraform.tfvars](https://labondemand.blob.core.windows.net/content/lab62680/resources/terraform.tfvars.sample)

**Task 2: Perform the single node HANA deployment**

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to initialize Terraform modules and provider plugins necessary to perform Terraform-based single-node HANA deployment:  
    
    >++++terraform init++++

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to identify changes to be performed by the Terraform-based single-node HANA deployment:  
       
    >++++terraform plan++++

1. [] In the Cloud Shell pane, from the Bash prompt, run the following to initiate Terraform-based single-node HANA deployment:  
      
    >++++terraform apply -auto-approve |& tee ~/clouddrive/output.txt++++

    > [!KNOWLEDGE] The deployment takes about 40 minutes to complete.



>[!note]In case the deployment fails in the middle of execution, run the following command or delete the resource group e.g. rsg-hanaholv1 from Azure Portal to start over.
>
>++++terraform destroy++++

===

#**Validate after deployment**

Saturday, January 11, 2020

6:17 PM

##**Exercise 2: Validate and remove the single node HANA deployment**

 

Duration: 20 minutes

In this exercise, you will validate the single-node HANA deployment you performed in the previous exercise by using the Windows bastion host. Once you successfully validate the deployment, you will remove all of its resources.

**Task 1: Connect to the single node HANA instance by using SAP HANA Studio**

1. [] From the lab computer, in the Azure portal, navigate to the blade of the hn1-win-bastion Azure VM operating as the Windows bastion host and initiate a Remote Desktop session. When prompted, sign in with the following credentials:

    -   Username: ++++.\bastion_user++++

    -   Password: ++++Demo@pass123++++

    > [!KNOWLEDGE] The user name of the Windows bastion host is set in /deploy/vm/modules/single_node_hana/variables.tf

    !IMAGE[Screenshot63.png](Screenshot63.png)

1. [] Within the Remote Desktop session to hn1-win-bastion, start Notepad, and open the hosts file located in *C:\Windows\System32\drivers\etc*.

1. [] Add the following entries to the host file, save your changes, and close the file:

    ```
    10.0.0.6        hn1-hdb0         
    ```
    > [!KNOWLEDGE]  10.0.0.6 is the private IP address assigned to the network interface of the Azure VM hosting the HANA instance.

1. [] Within the Remote Desktop session, start SAP HANA Studio Administration.

1. [] When prompted to select a workspace, accept the default value, and click Launch.

    !IMAGE[ao5ew3qy.jpg](ao5ew3qy.jpg)

1. [] When prompted to provide a password hint, click No.

    !IMAGE[iyze6egq.jpg](iyze6egq.jpg)

1. [] On the Overview page, click Open Administration Console.

    !IMAGE[Screenshot64.png](Screenshot64.png)

1. [] In the SAP HANA Administration Console, expand the Systems menu, and click Add System.

    !IMAGE[Screenshot65.png](Screenshot65.png)

1. [] In the Specify System dialog box, specify the following information, and click Next.

    -   **Host Name**: ++++hn1-hdb0++++

    -   **Instance number**: ++++01++++

    !IMAGE[kycptgb2.jpg](kycptgb2.jpg)

1. [] In the Connection Properties dialog box, select the Authentication by database user option, specify the following information, and click Finish.

    -   **User Name**: ++++SYSTEM++++

    -   **Password**: ++++Demo@pass123++++

    !IMAGE[utgy0t0n.png](utgy0t0n.png)

    >[!Alert]HANA “SYSTEM” user password gets locked after 3 unsuccessful attempts. 

1. [] Once you successfully connected to hn1-hdb0 as SYSTEM, select the HN1 (SYSTEM) node and click the Administration icon in the Systems toolbar and then click Open Default Administration.

    !IMAGE[Screenshot66.png](Screenshot66.png)

1. [] Review the Administration status on the Overview tab and ensure that all services are started.

    !IMAGE[v7sa3d0a.png](v7sa3d0a.png)

1. [] Switch to the Alerts tab, and verify they are no alerts indicating operational issues.

    !IMAGE[9trfdngz.png](9trfdngz.png)

**Task 2: Remove the single node HANA deployment**

1. [] Switch to the lab computer and, in the Cloud Shell pane, from the Bash prompt, run the following to change the current directory to the one hosting the Terraform and Ansible files that you used for the single node HANA deployment:  
       
    >++++cd ~/sap-hana/deploy/vm/modules/single_node_hana/++++
       
    > [!KNOWLEDGE] If needed, in the Azure portal, restart the Cloud Shell.

1. [] In the Cloud Shell pane, from the Bash prompt, run the followin to remove all resources provisioned.

    >++++terraform destroy -auto-approve++++
    
    > [!alert] If terraform destroy command doesn’t work, delete the resource group, rsg-hanaholv1 from Azure Portal.

1. [] When prompted, type yes and press the Enter key to continue with the removal of the deployed resources.

> [!KNOWLEDGE] Do not wait for the completion of the removal but instead proceed to the next exercise.


# Congratulations!

You have successfully completed this lab, to mark the lab as complete click **End**.
