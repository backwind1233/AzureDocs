<!-- Refer to https://github.com/Azure/azure-sdk-for-java/issues/35677  -->
# Integrate KeyVault JCA provider with Jarsigner

This guide provides a straightforward approach to integrating the KeyVault JCA provider with Jarsigner, ensuring a seamless process for users.

## Pre-requisites
Before beginning, ensure you have the following:

1. **Certificate in Azure KeyVault**: Prepare a certificate in Azure KeyVault.
2. **Service Principal**: Create a service principal.
3. **Permissions**: Assign the principal with read and write permissions for the KeyVault.
4. **Jarsigner Supported Algorithms**: Ensure you are using one of the following supported algorithms: DSA, RSA, or ECDSA.

## Step-by-Step Guide

Follow these steps carefully to achieve successful integration:

### Step 1: Configure JCA Provider Jar

1. **Download the JCA Provider Jar**: Obtain the JCA provider jar file from the [official](https://mvnrepository.com/artifact/com.azure/azure-security-keyvault-jca) source.
2. **Add to Java Environment**: 
    1. Place the jar under the folder `${JAVA_HOME}/jre/lib/ext`
        - ![Alt text](../Ressources/JCA/place_jar.png)

### Step 2: Set Up Environment Variables

Modify the `java.security` file in your Java installation to include the KeyVault JCA provider.

1. Open the file `java.security` in `${JAVA_HOME}/jre/lib/security`
   - ![Alt text](../Ressources/JCA/java_security.png)
1. Edit the file `java.security`
    1. Add `security.provider.${Input_Your_Number}=com.azure.security.keyvault.jca.KeyVaultJcaProvider` as the picture shows.
    1. ![Alt text](../Ressources/JCA/edit_provider.png)

### Step 3: Sign with Jarsigner

1. **Prepare Your Jar**: Have the jar file you wish to sign ready.
2. **Execute Jarsigner**: Use the Jarsigner tool with the KeyVault JCA provider to sign your jar file.
    1. Try to sign the jar using below command
         ```bash
         jarsigner   -keystore NONE -storetype AzureKeyVault \
                     -signedjar signerjar.jar ${replace_with_your_jar.jar} ${replace_with_certificate} \
                     -verbose  -storepass "" \
                     -providerName AzureKeyVault \
                     -providerClass com.azure.security.keyvault.jca.KeyVaultJcaProvider \
                     -J-Dazure.keyvault.uri=${replace_with_your_kv_uri} \
                     -J-Dazure.keyvault.tenant-id=${replace_with_your_sp_tenant-id} \
                     -J-Dazure.keyvault.client-id=${replace_with_your_sp_client-id} \
                     -J-Dazure.keyvault.client-secret=${replace_with_your_sp_client-secret} 
         ```
    1. The output may look like this
        - ![Alt text](../Ressources/JCA/output_1.png)
        - ![Alt text](../Ressources/JCA/output_2.png)

## Conclusion

By following these steps, you can easily integrate KeyVault JCA provider with Jarsigner. This method ensures a secure and efficient signing process using Azure KeyVault.

      
## Refs
- https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jarsigner.html

