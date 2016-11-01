### Secure(保护) Your Private Key
---
1. Create a file named keystore.properties (or anything you like) in the root
directory of your project. This file should contain your signing information,
as follows:
```
storePassword = myStorePassword
keyPassword = mykeyPassword
keyAlias = myKeyAlias
storeFile = myStoreFileLocation
```
2. In your module's build.gradle file, add code to load your keystore.properties file before the android {} block
``` java
// Create a variable called keystorePropertiesFile, and initialize it to your
// keystore.properties file, in the rootProject folder.
def keystorePropertiesFile = rootProject.file("keystore.properties")
// Initialize a new Properties() object called keystoreProperties.
def keystoreProperties = new Properties()
// Load your keystore.properties file into the keystoreProperties object.
keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
android {
    ...
}
```

 > Note: you could choose to store your keystore.properties file in another location
,In that case, you should modify the code above to correctly(正确的) initialize
keystorePropertiesFile using your actual(实际的)keystore.properties location

  ```
  e.g: Properties props = new Properties();
  props.load(new FileInputStream('app/keystore/liantong.properties'))
```
3. You can refer to properties stored in keystoreProperties
using the syntax(语法) keystoreProperties['propertyName']
Modefy the signingConfigs block of your module's build.gradle
file to reference the signing information stored in keystoreProperties using this syntax
```
android {
    signingConfigs {
        config {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
        }
    }
    ...
  }
```
4. select release build type,and click Build > Build APK to build your project.
and confirm that Android Studio has create a signed APK in the build/outputs/apk/directory for
your module.
> Because your build file no longer contain sensitive(敏感) information, you can now
include them in source control or upload them to shared codebase,Be sure to keep the
keystore.properties file secure.This may include
