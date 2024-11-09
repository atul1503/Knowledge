## Enable https in spring security

1. Generate a key pair and certificate using the `keytool` cli, which is provided by JDK itself.

   ```
     keytool -genkeypair -alias YOUR_ALIAS -keyalg RSA -keysize 4096 -storetype JKS -keystore KEYSTORE_NAME.jks -validity 3650 -storepass KEY_STORE_PASSWORD
   ```
   `-genkeypair`: creates a key pair and a certificate.
   `-alias YOUR_ALIAS`: an alias to be used for your keypair, but if there is only one key pair , it is not needed.
   `-keyalg RSA` : to specify the algorithm to generate key pair. Use RSA only.
   `4096`: size of key in bytes.
   `storetype JKS`: key store type. Key store is a file where your key pair and certificate is stored. There are multiple types but use JKS type only.
   `-keystore KEYSTORE_NAME.jks`: To specify the keystore file name. Your keystore will be generated with this file name. You need to keep this file in the `src/main/resources` folder. Remember to keep the extension as `.jks`.
   `-validity NO_OF_DAYS`: validity of the keystore file and its keys and certificates.
   `-storepass KEY_STORE_PASSWORD`: The password for the keystore itself.

   When you run this you will be asked to put name, organization etc, which you can press enter to ignore or put something if you like. But later, it will ask if the data entered is correct or not. Type `yes`.  Next it will ask for key password. Remember this password is different from keystore password. But I will suggest to keep it same. 


3. Move the `.jks` to `src/main/resources` folder.
4. In the `application.properties` file, put this:
   ```
    server.port = 8080
    server.ssl.key-store = classpath:KEYSTORE_NAME.jks
    server.ssl.key-store-password = KEY_STORE_PASSWORD
    server.ssl.key-alias = YOUR_ALIAS
   ```

The server.port is for specifying the port on which server will run.
key-store is to specify the path of keystore file.
key-store password is the password of key store itself.
key-alias to keep an alias for your key pair in the keystore file.

5. Start the server. Use `https://` when requesting.


