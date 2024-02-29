<!-- Refer to https://github.com/Azure/azure-sdk-for-java/issues/35677  -->
# Integrate KeyVault JCA provider with Jarsigner

This guide provides a straightforward approach to integrating the KeyVault JCA provider with Jarsigner, ensuring a seamless process for users.

## Prerequisites
Before beginning, ensure you have the following:

- An Azure subscription - [create one for free](https://azure.microsoft.com/free).
- [Java Development Kit (JDK)](/java/azure/jdk/) version 8 or higher.
- [Azure CLI](/cli/azure/install-azure-cli)
- Ensure you are using one of the following supported algorithms: DSA, RSA, or ECDSA.

## Step-by-Step Guide

Follow these steps carefully to achieve successful integration:

1. Prepare your parameters
```shell
DATE_STRING=$(date +%H%M%S)
RESOURCE_GROUP_NAME=jarsigner-rg-$DATE_STRING
KEYVAULT_NAME=jarsiner-kv-$DATE_STRING
CERT_NAME=jarsiner-cert-$DATE_STRING
SERVICE_PRINCIPAL_NAME=jarsiner-sp-$DATE_STRING
```
2. Create a resource group

```shell
az group create --name $RESOURCE_GROUP_NAME --location "EastUS"
```

3. Create a key vault

```shell
az keyvault create --name $KEYVAULT_NAME --resource-group $RESOURCE_GROUP_NAME --location "EastUS"
```

4. Get the key vault uri

```shell
KEYVAULT_URL=$(az keyvault show --name $KEYVAULT_NAME --query "properties.vaultUri" --resource-group $RESOURCE_GROUP_NAME -o tsv| tr -d '\r\n')
echo $KEYVAULT_URL
```
Note the output as kv_uri for later use.

5. Add a certificate to Key Vault

```shell
az keyvault certificate create --vault-name $KEYVAULT_NAME -n $CERT_NAME -p "$(az keyvault certificate get-default-policy)"
```

6. Create a Service Principal

```shell
SP_JSON=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME)

CLIENT_ID=$(echo $SP_JSON | jq -r '.appId')
CLIENT_SECRET=$(echo $SP_JSON | jq -r '.password')
TENANT=$(echo $SP_JSON | jq -r '.tenant')

echo "CLIENT_ID:"$CLIENT_ID
echo "CLIENT_SECRET:"$CLIENT_SECRET
echo "TENANT:"$TENANT

```
Note the appId and password from the output, you'll need them later.

7. Get the objectId

```shell
OBJECTID=$(az ad sp show --id "$CLIENT_ID" --query id -o tsv | tr -d '\r\n')
echo $OBJECTID
```

7. Assign Permissions to Service Principal:

```shell
az keyvault set-policy --name $KEYVAULT_NAME --resource-group $RESOURCE_GROUP_NAME --object-id $OBJECTID --secret-permissions get 

az keyvault set-policy --name $KEYVAULT_NAME --resource-group $RESOURCE_GROUP_NAME --object-id $OBJECTID --certificate-permissions get list
```


### Step 1: Configure JCA Provider Jar

1. **Download the JCA Provider Jar**: Obtain the JCA provider jar file from the [official](https://mvnrepository.com/artifact/com.azure/azure-security-keyvault-jca) source.
2. **Add to Java Environment**: 
    1. Place the jar under the folder `${JAVA_HOME}/jre/lib/ext`
        - ![Alt text](../Ressources/JCA/place_jar.png)

### Step 2: Sign with Jarsigner

1. **Prepare Your Jar**: Have the jar file you wish to sign ready.
2. **Execute Jarsigner**: Use the Jarsigner tool with the KeyVault JCA provider to sign your jar file.
    1. Try to sign the jar using below command
         ```bash
         jarsigner   -keystore NONE -storetype AzureKeyVault \
                     -signedjar signerjar.jar demo.jar "${CERT_NAME}" \
                     -verbose  -storepass "" \
                     -providerName AzureKeyVault \
                     -providerClass com.azure.security.keyvault.jca.KeyVaultJcaProvider \
                     -J--module-path="kvjca.jar" \
                     -J--add-modules="com.azure.security.keyvault.jca" \
                     -J-Dazure.keyvault.uri=${KEYVAULT_URL} \
                     -J-Dazure.keyvault.tenant-id=${TENANT} \
                     -J-Dazure.keyvault.client-id=${CLIENT_ID} \
                     -J-Dazure.keyvault.client-secret=${CLIENT_SECRET} 
         ```
    1. The output may look like this
        - ![Alt text](../Ressources/JCA/output_1.png)
        - ![Alt text](../Ressources/JCA/output_2.png)

## Conclusion

By following these steps, you can easily integrate KeyVault JCA provider with Jarsigner. This method ensures a secure and efficient signing process using Azure KeyVault.

## Clean up resources
To avoid Azure charges, you should clean up unnecessary resources.  

```shell
az group delete --name $RESOURCE_GROUP_NAME --yes --no-wait
az ad app delete --id $CLIENT_ID
```

## Refs
- https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jarsigner.html

