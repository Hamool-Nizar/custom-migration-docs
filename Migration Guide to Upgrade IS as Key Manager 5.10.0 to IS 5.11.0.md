# Upgrading WSO2 IS as Key Manager to 5.11.0

The following information describes how to upgrade your WSO2 API Manager (WSO2 API-M) environment from API-M 3.2.0 to 4.1.0 when WSO2 Identity Server (WSO2 IS) is the Key Manager in the pre-migrated setup.

>Step A - Upgrade IS as Key Manager 5.10.0 to IS 5.11.0
>
>Step B - Upgrade API Manager 3.2.0 to 4.1.0

### Step A - Upgrade IS as Key Manager 5.10.0 to IS 5.11.0

Follow step 1 to step 3 below to upgrade your IS as Key Manager 5.10.0 to IS 5.11.0
>- Step 1 - Migrate the IS as KM configurations to IS 5.11.0
>- Step 2 - Migrate the IS as KM resources to IS 5.11.0
>- Step 3 - Migrate the IS as KM components to IS 5.11.0

#### Step 1 - Migrate the IS as KM configurations

1. Download WSO2 IS 5.11.0 distribution from [here](https://wso2.com/identity-and-access-management/) and extract it. `<IS_HOME>` refers to the root folder of the extracted WSO2 IS 5.11.0.

2. Add following configurations in the `<IS_HOME>/repository/conf/deployment.toml` file.


>**deployment.toml**
>```
>[[event_listener]]
>id = "token_revocation"
>type = "org.wso2.carbon.identity.core.handler.AbstractIdentityHandler"
>name = "org.wso2.is.notification.ApimOauthEventInterceptor"
>order = 1
>
>[[resource.access_control]]
>context = "(.)/keymanager-operations/user-info/claims(.)"
>secure = true
>http_method = "GET"
>permissions = "/permission/admin/manage/identity/usermgt/list"
>scopes = "internal_user_mgt_list"
>
>[[resource.access_control]]
>context = "(.*)/keymanager-operations/user-info/claims/generate"
>secure = true
>http_method = "POST"
>permissions = "/permission/admin/manage/identity/usermgt/list"
>scopes = "internal_user_mgt_list"
>
>[[resource.access_control]]
>context = "(.*)/keymanager-operations/dcr/register"
>secure = true
>http_method = "POST"
>permissions = "/permission/admin/manage/identity/applicationmgt/create"
>scopes = "internal_application_mgt_create"
>
>[[resource.access_control]]
>context = "(.*)/keymanager-operations/dcr/register(.*)"
>secure = true
>http_method = "GET"
>permissions = "/permission/admin/manage/identity/applicationmgt/view"
>scopes = "internal_application_mgt_view"
>
>[[resource.access_control]]
>context = "(.*)/keymanager-operations/dcr/register(.*)"
>secure = true
>http_method = "PUT"
>permissions = "/permission/admin/manage/identity/applicationmgt/update"
>scopes = "internal_application_mgt_update"
>
>[[resource.access_control]]
>context = "(.*)/keymanager-operations/dcr/register(.*)"
>secure = true
>http_method = "DELETE"
>permissions = "/permission/admin/manage/identity/applicationmgt/delete"
>scopes = "internal_application_mgt_delete"
>
>[tenant_context.rewrite]
>custom_webapps = ["/keymanager-operations/"]
>```

3. Configure the event listener endpoint to publish controller events to the Traffic Manager.
    ```
    [event_listener.properties]
    notification_endpoint = "https://<tm.wso2.com>:9443/internal/data/v1/notify"
    username = "${admin.username}"
    password = "${admin.password}"
    'header.X-WSO2-KEY-MANAGER' = "WSO2-IS"
    ```

4. IS 5.11.0 uses Symmetric encryption as default and API-M 4.1.0 uses Asymmetric algorithm as default. So when using IS 5.11.0 as KM, if you had used Asymmetric encryption in the previous API-M version, please add below configurations to deployment.toml.
    ```
    [keystore]
    userstore_password_encryption = "InternalKeyStore"

    [system.parameter]
    "org.wso2.CipherTransformation"="RSA/ECB/OAEPwithSHA1andMGF1Padding"

    [encryption]
    internal_crypto_provider = "org.wso2.carbon.crypto.provider.KeyStoreBasedInternalCryptoProvider"
    ```
5. Migrate IS KM 5.10.0 configurations as per the instructions in **Migrating the configurations**.

>**Important**
>
>When following the instructions of IS 5.11.0 migration guide, make sure to follow the below guidelines as well.
>- Configure the identity_db datasource in `<IS_HOME>/repository/conf/deployment.toml` of IS 5.11.0 by pointing to the old WSO2AM_DB.
>```
>[database.identity_db]
>type = "mysql"
>url = "jdbc:mysql://localhost:3306/am_db"
>username = "wso2carbon"
>password = "wso2carbon"
>```
>- If you have used a JDBCUserStoreManager as the userstore in previous IS as KM setup, comment/remove the following from `<IS_HOME>/repository/conf/deployment.toml` in IS 5.11.0.
>```
>#[user_store]
>#type = "read_write_ldap_unique_id"
>#connection_url = "ldap://localhost:${Ports.EmbeddedLDAP.LDAPServerPort}"
>#connection_name = "uid=admin,ou=system"
>#connection_password = "admin"
>#base_dn = "dc=wso2,dc=org"
>```
>- Instead, add the following to the <IS_HOME>/repository/conf/deployment.toml in IS 5.11.0
>```
>[user_store]
>type = "database_unique_id"
>```
>- If the user store type in the previous version is set to database instead of default database_unique_id, update `<IS_HOME>/repository/conf/deployment.toml` file as follows
>```
>[user_store]
>type = "database"
>```
>- If you have used separate DBs for user management and registry in the previous IS as KM version, you need to configure WSO2REG_DB and WSO2USER_DB databases separately in `<IS_HOME>/repository/conf/deployment.toml` of IS 5.11.0 to avoid any issues.
>```
>[database.shared_db]
>type = "mysql"
>url = "jdbc:mysql://localhost:3306/reg_db"
>username = "wso2carbon"
>password = "wso2carbon"
>
>[database.user]
>type = "mysql"
>url = "jdbc:mysql://localhost:3306/um_db"
>username = "wso2carbon"
>password = "wso2carbon"
>```
>- If you have used a separate user management database as the primary userstore in previous IS as KM setup, add the following to the `<IS_HOME>/repository/conf/deployment.toml` in IS 5.11.0.
>```
>[realm_manager]
>data_source = "WSO2USER_DB"
>```
>You DO NOT NEED to copy the API-M Key Manager specific configurations from `<OLD_IS_KM_HOME>/repository/conf/api-manager.xml` of previous IS as KM version to IS 5.11.0.


#### Step 2 - Migrate the IS as KM Resources

1. Contact the WSO2 Support Team to get the WSO2 IS Connector.

2. Extract the distribution and copy the following JAR files to the `<IS_HOME>/repository/components/dropins` directory.
    - wso2is.key.manager.core-1.4.2.jar
    - wso2is.notification.event.handlers_1.4.2.jar

3. Add keymanager-operations.war from the extracted distribution to the `<IS_HOME>/repository/deployment/server/webapps` directory.

4. Make sure you have met the following prerequisites before proceeding with the instructions to migrate to 5.11.0.

    1. Make sure you satisfy the prerequisites in [Before you begin](../../setup/migration-guide).
    2. Carry out the pre-processing steps in the **Preparing for migration** document.
   
5. Once all the above prerequisites have been met, follow the instructions given below to migrate to the latest version.
   1. If you have manually added JAR files to the `<OLD_IS_HOME>/repository/components/lib` directory, copy and paste those JARs into the `<NEW_IS_HOME>/repository/components/lib` directory.
   2. Copy the `.jks` files from the `<OLD_IS_HOME>/repository/resources/security` directory and paste them in the `<NEW_IS_HOME>/repository/resources/security` directory.
        > **Note**
        >
        > In WSO2 Identity Server 5.11.0, it is required to use a certificate with an RSA key size greater than 2048. If you used a certificate with a weak RSA key (key size less than 2048) in the previous IS version, add the following configuration to `<NEW_IS_HOME>/repository/conf/deployment toml` to configure internal and primary key stores.
        >
        >   ```toml
        >   [keystore.primary]
        >   file_name = "primary.jks"
        >   type = "JKS"
        >   password = "wso2carbon"
        >   alias = "wso2carbon"
        >   key_password = "wso2carbon"
        >
        >   [keystore.internal]
        >   file_name = "internal.jks"
        >   type = "JKS"
        >   password = "wso2carbon"
        >   alias = "wso2carbon"
        >   key_password = "wso2carbon"
        >   ```
        >
        > Make sure to point the internal keystore to the keystore copied from the previous WSO2 Identity Server version. The primary keystore can be pointed to a keystore with a certificate with a strong RSA key.
   3. Ensure that you have migrated the configurations into the new version as advised in the document on **preparing for migration** document.
   4. Make sure that all the properties in the `<IS_HOME>/repository/conf/deployment.toml` file, such as the database configurations, are set to the correct values based on the requirement.

#### Step 3 - Migrate the IS as KM Components

1. Make sure you backed up all the databases in API-M 3.2.0

2. Download the identity component migration resources and unzip it in a local directory.
    Contact the WSO2 Support Team to get the `wso2is-migration-x.x.x.zip` file.
    Let's refer to this directory that you downloaded and extracted as <IS_MIGRATION_TOOL_HOME>.

3. Copy the `migration-resources` folder from the extracted folder to the `<IS_HOME> `directory.

4. Open the `migration-config.yaml` file in the `migration-resources` directory and make sure that the currentVersion element is set to 5.10.0, as shown below.
    ```
    migrationEnable: "true"
    currentVersion: "5.10.0"
    migrateVersion: "5.11.0"
    ```

5. Remove the following 2 steps from migration-config.yaml which is included under version: "5.10.0"
    ```
    - version: "5.10.0"
        migratorConfigs:
        -
            name: "MigrationValidator"
            order: 2
        -
            name: "SchemaMigrator"
            order: 5
            parameters:
            location: "step2"
            schema: "identity"
        -
            name: "TenantPortalMigrator"
            order: 11
    ```
6. Remove the following 2 steps from `migration-config.yaml` which is included under version: "5.11.0"
    ```
    -
        name: "EncryptionAdminFlowMigrator"
        order: 1
        parameters:
        currentEncryptionAlgorithm: "RSA/ECB/OAEPwithSHA1andMGF1Padding"
        migratedEncryptionAlgorithm: "AES/GCM/NoPadding"
        schema: "identity"
    -
        name: "EncryptionUserFlowMigrator"
        order: 2
        parameters:
        currentEncryptionAlgorithm: "RSA/ECB/OAEPwithSHA1andMGF1Padding"
        migratedEncryptionAlgorithm: "AES/GCM/NoPadding"
        schema: "identity"
    ```

7. Copy the `org.wso2.carbon.is.migration-x.x.x.jar` from the `<IS_MIGRATION_TOOL_HOME>/dropins` directory to the `<IS_HOME>/repository/components/dropins` directory.

>**Important**
>
>In WSO2 Identity Server 5.11.0, groups include user store roles and roles include internal roles. Therefore, from IS 5.11.0 onwards, we cannot have the same admin role in both primary and internal user domains. If the same admin role exists in both UM domains of your older version, we should add the PRIMARY/admin to the internal domain with a different admin role name during the group role separation. To do that, you have to follow the below steps.
>- Add the following configuration to `<IS_HOME>/repository/conf/deployment.toml` file.Rename the admin role with a new role name. This can be any value you prefer.
>```
>[super_admin]
>admin_role = "<NEW-ADMIN-ROLE-NAME>"
>create_admin_account = false
>```
>- Open the migration-config.yaml file in the migration-resources directory and add the admin role name of the current version to the parameter `currentAdminRoleName` under the `GroupsAndRolesMigrator`. For example, "admin" which is the default admin role name.
>```
>name: "GroupsAndRolesMigrator"
>    order: 4
>    parameters:
>        currentAdminRoleName:"<CURRENT-ADMIN-ROLE_NAME>"
>```

8. Modify create_admin_account to false in `<IS_HOME>/repository/conf/deployment.toml` if it is not changed already.
    ```
    [super_admin]
    create_admin_account = false
    ```

9. Start WSO2 IS 5.11.0 as follows to carry out the complete Identity component migration.

    **Linux / Mac OS**
    ```
    sh wso2server.sh -Dmigrate -Dcomponent=identity
    ```
    **Windows**
    ```
    wso2server.bat -Dmigrate -Dcomponent=identity
    ```


>**Warning**
>
>Depending on the number of records in the identity tables, this identity component migration will take a considerable amount of time to finish. Do NOT stop the server during the migration process and please wait until the migration process finish completely and server get started.

10. After you have successfully completed the migration, stop the server and remove the following files and folders.
    - Remove the org.wso2.carbon.is.migration-x.x.x.jar file, which is in the `<IS_HOME>/repository/components/dropins` directory.
    - Remove the `migration-resources` directory, which is in the `<IS_HOME>` directory.
    - If you ran WSO2 IS as a Windows Service when doing the IS migration , then you need to remove the following parameters in the command line arguments section (CMD_LINE_ARGS) of the wso2server.bat file.
    ```
    -Dmigrate -Dcomponent=identity
    ```

### Step B - Upgrade API Manager 3.2.0 to 4.1.0

Follow the steps mentioned in Upgrading API-M from 3.2.0 to 4.1.0 to upgrade your API-M environment from 3.2.0 to 4.1.0.


>**Important**
>
>- When following guidelines under **Step 1 - Migrate the API Manager configurations**, make sure to add the following to the `<API-M_HOME>/repository/conf/deployment.toml`. This is to configure your IS 5.11.0 as the Resident Key Manager of your API-M 4.1.0 deployment.
>   ```
>   [apim.key_manager]
>   service_url ="https://<IS_5.11.0_HOST_NAME>:<PORT>/services/"
>   type = "WSO2-IS"
>   ```
>   - Do NOT copy any other Key Manager specific configurations coming from previous API-M version to the latest pointing to the IS instance.
>
>- SKIP guidelines under **Step 3 - Migrate the Identity Components**
>   - You have already done this in Step 3 of **Step A - Upgrade IS as Key Manager 5.10.0 to IS 5.11.0**.
>- After configuring WSO2 IS 5.11.0 as the Resident Key Manager and before starting the API-M 4.1.0 server for the first time in Step 6 under **Step 6 - Restart the WSO2 API-M 4.1.0 Server**, make sure you have already started WSO2 IS 5.11.0.


