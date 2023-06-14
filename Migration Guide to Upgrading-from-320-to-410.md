# Upgrading API Manager from 3.2.0 to 4.1.0


The following information describes how to upgrade your API Manager server **from API-M 3.2.0 to 4.1.0**.

## Prerequisites

1. Review what has been changed in this release. See the **What Has Changed** page.

2. Before you migrate, follow the **Upgrading Guidelines** document to get an understanding of the migration process.

3. Download [WSO2 API Manager 4.1.0](http://wso2.com/api-management/) and unzip it in the <API-M_4.1.0_HOME> directory.

4. Update API-M 4.1.0 to the latest U2 update level.
    
Follow the instructions below to upgrade your WSO2 API Manager server **from WSO2 API-M 3.2.0 to 4.1.0**.

## Steps to migrate to WSO2 API-M 4.1.0
- [Upgrading API Manager from 3.2.0 to 4.1.0](#upgrading-api-manager-from-320-to-410)
  - [Prerequisites](#prerequisites)
  - [Steps to migrate to WSO2 API-M 4.1.0](#steps-to-migrate-to-wso2-api-m-410)
    - [Step 1 - Migrate the API Manager configurations](#step-1---migrate-the-api-manager-configurations)
    - [Step 2: Migrate the API Manager Resources](#step-2-migrate-the-api-manager-resources)
    - [Step 3: Migrate the Identity Components](#step-3-migrate-the-identity-components)
    - [Step 4: Migrate the API Manager Components](#step-4-migrate-the-api-manager-components)
    - [Step 5: Re-Index the API Manager artifacts](#step-5-re-index-the-api-manager-artifacts)
    - [Step 6 - Restart the WSO2 API-M 4.1.0 Server](#step-6---restart-the-wso2-api-m-410-server)

### Step 1 - Migrate the API Manager configurations

> **Warning**
>
>    Do not copy entire configuration files from the current version of WSO2 API Manager to the new one, as some configuration files may have changed. Instead, redo the configuration changes in the new configuration files.

1. Open the `<API-M_4.1.0_HOME>/repository/conf/deployment.toml` file and configure the datasource configurations for the available databases in the API-M 3.2.0 API-M set up to migrate them to API-M-4.1.0

    1. API Manager database (WSO2AM_DB)

        Ex: 
        ```
        [database.apim_db]
        type = "mysql"
        url = "jdbc:mysql://localhost:3306/am_db"
        username = "username"
        password = "password"
        ```

    2. Shared database/s (WSO2_SHARED_DB)
        
        Ex:
        ```
        [database.shared_db]
        type = "mysql"
        url = "jdbc:mysql://localhost:3306/shared_db"
        username = "username"
        password = "password"
        ```

        1. Registry database
            
            Ex:
            ```
            [database.shared_db]
            type = "mysql"
            url = "jdbc:mysql://localhost:3306/reg_db"
            username = "username"
            password = "password"
            ```
            
            - Config DB  (WSO2CONFIG_DB)
                
                Ex:

                ```
                [database.config]
                type = "mysql"
                url = "jdbc:mysql://localhost:3306/config_db"
                username = "username"
                password = "password"
                ```
    
        2. User Stores (WSO2UM_DB) 
        
            Ex:
            ```
            [database.user]
            type = "mysql"
            url = "jdbc:mysql://localhost:3306/um_db"
            username = "username"
            password = "password"
            ```    

    > **If you are using another DB type**
    >
    >    If you are using DB type other than **H2** or **MySQL** or **Oracle**, you need to add the `driver` and `validationQuery` parameter configuration additionally as given below.
    >
    >    **MSSQL**
    >    ```sql
    >    [database.apim_db]
    >    type = "mssql"
    >    url = "jdbc:sqlserver://localhost:1433;databaseName=mig_am_db;>SendStringParametersAsUnicode=false"
    >    username = "username"
    >    password = "password"
    >    driver = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
    >    validationQuery = "SELECT 1"
    >    ```
    >
    >    **PostgreSQL**
    >    ```sql
    >    [database.apim_db]
    >    type = "postgre"
    >    url = "jdbc:postgresql://localhost:5432/mig_am_db"
    >    username = "username"
    >    password = "password"
    >    driver = "org.postgresql.Driver"
    >    validationQuery = "SELECT 1"
    >    ```
    >
    >    **Oracle**
    >    ```sql
    >    [database.apim_db]
    >    type = "oracle"
    >    url = "jdbc:oracle:thin:@localhost:1521/mig_am_db"
    >    username = "username"
    >    password = "password"
    >    driver = "oracle.jdbc.driver.OracleDriver"
    >    validationQuery = "SELECT 1 FROM DUAL"
    >    ```    

2. If you have used separate DB for user management, you need to update `<API-M_4.0.0_HOME>/repository/conf/deployment.toml` file as follows, to point to the correct database for user management purposes.
     
    ```toml
    [realm_manager]
    data_source = "WSO2USER_DB"
    ```

3. Modify the `[apim.gateway.environment]` tag in the `<API-M_HOME>/repository/conf/deployment.toml` file, the name should change to "Production and Sandbox". By default, it is set as `Default` in API Manager 4.1.0.
    
    ```toml
    [[apim.gateway.environment]]
    name = "Production and Sandbox"
    ```
    Modify the `[apim.sync_runtime_artifacts.gateway]` tag in the `<API-M_HOME>/repository/conf/deployment.toml`, so that the value of `gateway_labels` should be the name of old gateway environment (old default one is "Production and Sandbox") or we need to add the old one as a new gateway environment, while the new current default label (current default one is "Default") remains as it is.

    ```toml
    [apim.sync_runtime_artifacts.gateway]
    gateway_labels = ["Production and Sandbox", "Default"]
    ```
    or
    ```toml
    [apim.sync_runtime_artifacts.gateway]
    gateway_labels = ["Production and Sandbox"]
    ```

    This config defines an array of the labels that the Gateway is going to subscribe to. Only the APIs with these labels will be pulled from the extension point and deployed.

    > **Info**
    >
    >    If you have changed the name of the gateway environment in your older version, then when migrating, make sure that you change the `[apim.gateway.environment]` tag  and `[apim.sync_runtime_artifacts.gateway]` tag  accordingly. For example, if your gateway environment was named `Test` in the `<OLD_API-M_HOME>/repository/conf/api-manager.xml` file, you have to change the toml config as shown below.
    >
    >    ```toml
    >    [[apim.gateway.environment]]
    >    name = "Test"
    >    ``` 
    >
    >    ```toml
    >    [apim.sync_runtime_artifacts.gateway]
    >    gateway_labels = ["Test"]
    >    ```

4. Make sure to add the following to the `<API-M_HOME>/repository/conf/deployment.toml`. This is to configure your IS 5.11.0 as the Resident Key Manager of your API-M 4.1.0 deployment.
    >   ```
    >   [apim.key_manager]
    >   service_url ="https://<IS_5.11.0_HOST_NAME>:<PORT>/services/"
    >   type = "WSO2-IS"
    >   ```
    - Do NOT copy any other Key Manager specific configurations coming from previous API-M version to the latest pointing to the IS instance.

5. Run the following script on the registry database to add missing registry indices. Use the script that corresponds to your DB type.

    1_DB2.db2

    2_MSSQL.sql

    3_MySQL.sql
   
    4_Oracle.sql
    
    5_PostgreSQL.sql
   
6. If you have enabled any other feature related configurations at `<API-M_4.1.0_HOME>/repository/conf/deployment.toml`, make sure to add them in to `<API-M_4.1.0_HOME>/repository/conf/deployment.toml` file.

### Step 2: Migrate the API Manager Resources

Follow the instructions below to migrate existing API Manager resources from the current environment to API-M 4.1.0.

1.  Copy the relevant JDBC driver to the `<API-M_4.1.0_HOME>/repository/components/lib` folder.

2.  Copy the keystores (i.e., `client-truststore.jks`, `wso2cabon.jks` and any other custom JKS) used in the previous version and replace the existing keystores in the `<API-M_4.1.0_HOME>/repository/resources/security` directory to persist the information about the added private keys, certificates and the list of trusted CA that have been used in API-M_3.2.0. If you wish to add WSO2 IS 5.11.0 as the Resident Key Manager in API-M 4.1.0 new deployment, you have to copy the same keystores in to `<IS_5.11.0_HOME>/repository/resources/security` directory.

    > **If you have enabled Secure Vault**
    >
    >    If you have enabled secure vault in the previous API-M version, you need to add the property values again according to the new config modal and run the script as below. Refer [Encrypting Passwords in Configuration files](https://apim.docs.wso2.com/en/4.1.0/install-and-setup/setup/security/logins-and-passwords/working-with-encrypted-passwords) for more details.
    >
    >    **Linux**
    >    ```
    >    ./ciphertool.sh -Dconfigure
    >    ```
    >
    >    **Windows**
    >    ```
    >    ./ciphertool.bat -Dconfigure
    >    ``` 

3. If you have used global sequences in the previous version, please copy the sequence files to `<PRODUCT_HOME>/repository/deployment/server/synapse-configs/default/sequences` folder and add the below config to `deployment.toml` file to prevent the sequence files from getting removed from the file system on server startup.

    **Format**
    ```
    [apim.sync_runtime_artifacts.gateway.skip_list]
    sequences = [<SEQUENCE FILES LIST HERE>]
    ```

    **Example**
    ```
    [apim.sync_runtime_artifacts.gateway.skip_list]
    sequences = ["WSO2AM--Ext--In.xml"]
    ```

### Step 3: Migrate the Identity Components

You have already done this in Step 3 of **Step A - Upgrade IS as Key Manager 5.10.0 to IS 5.11.0**. Therefore, you can skip this step and continue to the Step 4

### Step 4: Migrate the API Manager Components

You have to run the following migration client to update the API Manager artifacts.

1. Contact the WSO2 Support Team to obtain the migration-resources.zip]file. Copy the extracted migration-resources to the `<API-M_4.1.0_HOME>` folder.

2. Obtain the [org.wso2.carbon.apimgt.migrate.client-4.1.0.x.jar file from the WSO2 Support Team and copy it into the `<API-M_4.1.0_HOME>/repository/components/dropins` directory.

3. Prior to API-M migration, run the below command to execute the pre-migration step that will validate your old data.

    **Pre-validators**
        
    | API Validators                            |                                          |                                      |
    | ----------------------------------------- | ---------------------------------------- | ------------------------------------ |
    | **CLI Tag**                               | **Pre-validator**                        | **Purpose**                          |
    | `apiDefinitionValidation`                 | API Definition Validator                 | Validates if the API definitions are up to standards so that issues are not encountered during migration. Validations are done to check if APIs have valid OpenAPI, WSDL, Streaming API, or GraphQL API definitions. |
    | `apiAvailabilityValidation`               | API Availability Validator               | Validates the API availability in the database with respect to the API artifacts in the registry in order to verify there are no corrupted entries in the registry. |
    | `apiResourceLevelAuthSchemeValidation`    | API Resource Level Auth Scheme Validator | Usage of resource level security with `Application` and `Application User` in 2.x versions, is not supported. This pre-validation checks and warns about such APIs with unsupported resource-level auth schemes. |
    | `apiDeployedGatewayTypeValidation`        | API Deployed Gateway Type Validator      | If the deployed Gateway type of an API is given as `none`, deployment of that API will be skipped at migration. This pre-validation warns on such APIs having deployed Gateway type as `none`. |

    | Application Validators                    |                                          |                                      |
    | ----------------------------------------- | ---------------------------------------- | ------------------------------------ |
    | **CLI Tag**                               | **Pre-validator**                        | **Purpose**                          |
    | `appThirdPartyKMValidation`               | Third Party Key Manager Usage Validator  | If third party key managers were used with the old API-M, they may need to be reconfigured for the new API-M verison. This prevalidation checks the usage of the built-in key manager and warns otherwise. |

    In this step, you can run data validation on all the existing validators or selected validators. If you only use the `-DrunPreMigration` command, all existing validations will 
    be enabled. If not, you can provide a specific validator, such as  `-DrunPreMigration=apiDefinitionValidation`, which only validates the API definitions.

    **Linux / Mac OS**
    ```bash
    sh api-manager.sh -Dmigrate -DmigrateFromVersion=3.2.0 -DmigratedVersion=4.1.0 -DrunPreMigration
    ```

    **Windows**
    ```
    api-manager.bat -Dmigrate -DmigrateFromVersion='3.2.0' -DmigratedVersion='4.1.0' 
    -DrunPreMigration
    ```

    > **If you want to save the invalid API definitions**
    >
    > You can save the invalid API definitions to the local file system during this data validation step if required. Use the `-DsaveInvalidDefinition` option for this as follows. The invalid definitions will be stored under a folder named `<API-M_4.1.0_HOME>/invalid-swagger-definitions` in the form of `<API_UUID>.json`. Then you can manually correct these definitions.
    >
    >   **Linux / Mac OS**
    >   ```bash
    >   sh api-manager.sh -Dmigrate -DmigrateFromVersion=3.2.0 -DmigratedVersion=4.1.0 -DrunPreMigration -DsaveInvalidDefinition
    >    ```
    >
    >   **Windows**
    >   ```bash
    >   api-manager.bat -Dmigrate -DmigrateFromVersion='3.2.0' -DmigratedVersion='4.1.0' -DrunPreMigration -DsaveInvalidDefinition
    >    ```

    Check the server logs and verify if there are any errors logs. If you have encountered any errors in the API definitions, you have to correct them manually on the old version before proceeding to step 4.

4.  Start the API-M server to migrate the API-M components as follows.

    **Linux / Mac OS**
    ```bash
    sh api-manager.sh -Dmigrate -DmigrateFromVersion=3.2.0 -DmigratedVersion=4.1.0
    ```
    
    **Windows**
    ```bash
    api-manager.bat -Dmigrate -DmigrateFromVersion=3.2.0 -DmigratedVersion=4.1.0
    ```

5.  Shutdown the API-M server.
    
    -   Remove the `org.wso2.carbon.apimgt.migrate.client-4.1.0.x.jar` file, which is in the `<API-M_4.1.0_HOME>/repository/components/dropins` directory.

    -   Remove the `migration-resources` directory, which is in the `<API-M_4.1.0_HOME>` directory.

    > **Note**
    >
    >    Make sure you have copied the tenant's userstores if you have configured them in WSO2 API Manager 3.2.0.

### Step 5: Re-Index the API Manager artifacts

1. To re-index the API artifacts in the registry, Add the following configuration into the `<API-M_4.1.0_HOME>/repository/conf/deployment.toml` file.
        
    ```toml
    [indexing]
    re_indexing = 1
    ```
        
    Note that you need to increase the value of `re_indexing` by one each time you need to re-index.

             
2. If the `<API-M_4.1.0_HOME>/solr` directory exists, take a backup and thereafter delete it. 

    > **Important**
    >
    >    If you use a clustered/distributed API Manager setup, do the above change in `deployment.toml` of Publisher and Devportal nodes. Make sure to keep a delay between nodes to execute this step to re-index each node, as the database can experience a large load.

    > **Note**
    >
    >    Note that it takes a considerable amount of time for the API Manager to re-index the artifacts, depending on the API count and the number of tenants.

### Step 6 - Restart the WSO2 API-M 4.1.0 Server

- Before starting the API-M 4.1.0 server for the first time make sure you have already started WSO2 IS 5.11.0.
    
    **Linux / Mac OS**
    ```
    sh wso2server.sh
    ```
    **Windows**
    ```
    wso2server.bat
    ```

- Then Restart the APIM 4.1.0 server.

    **Linux / Mac OS**
    ```bash
    sh api-manager.sh
    ```

    **Windows**
    ```bash
    api-manager.bat
    ```

This concludes the upgrade process.