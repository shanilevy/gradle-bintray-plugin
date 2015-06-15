# Overview

The Gradle Bintray Plugin allows you to publish artifacts to a repository on [bintray](https://bintray.com/).

The plugin adds the `bintrayUpload` task to your projects, which allows you to upload to bintray and optionally create
the target package and version. Artifacts can be uploaded from the specified configurations or (the newly supported) publications

[ ![Download](https://api.bintray.com/packages/jfrog/jfrog-jars/gradle-bintray-plugin/images/download.svg) ](https://bintray.com/jfrog/jfrog-jars/gradle-bintray-plugin/_latestVersion)

# Getting Started Using the Plugin
The following steps add the Gradle Bintray Plugin to your Gradle build script.

#### Step 1: [Sign up](https://bintray.com/docs/usermanual/working/working_allaboutjoiningbintraysigningupandloggingin.html) and define user and key for [bintray](https://bintray.com/)

#### Step 2: Add Bintray Plugin to your Gradle build script. 

Depending on the version of Gradle you're running, there are different usage scenarios. Add one of the following snippets to your `build.gradle` file:

###### Gradle >= 2.1
```groovy
plugins {
    id "com.jfrog.bintray" version "1.2"
}
```

###### Gradle < 2.1
```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
    }
}
apply plugin: 'com.jfrog.bintray'
```
* make sure the plugin exists for every sub-project

**Gradle Compatibility:**
When using Gradle publications or when using `filesSpec` for direct file uploads, you'll need to use Gradle 2.x; Otherwise, the plugin is compatible with Gradle 1.12 and above.

 **JVM Compatibility:**
Java 6 and above.

#### Step 3: Add the "bintray" closure to your `build.gradle` file.

Add the bellow "bintray" closure, adding your bintray user name and key.

You can do it either explicitly:

```groovy
bintray {
    user = 'bintray_user'
    key = 'bintray_api_key'	
}
```

Or by using environment variables: 'BINTRAY_USER' and 'BINTRAY_KEY':

```groovy
bintray {
    user = System.getenv('BINTRAY_USER')
    key = System.getenv('BINTRAY_KEY')	
}
```

#### Step 4: Define the publications closure in your `build.gradle` file.

Please advise that this is currently working only with Maven Publications.

This can be done either by defining maven POM parameters:

```groovy
publishing {
	publications {
		mavenJava(MavenPublication) {
			from components.java
			groupId 'org.jfrog.gradle.sample'
			artifactId 'gradle-project'
			version '1.1'
		}
	}
}
```

Or by taking the POM parameters directly from the project definition:

```groovy
allprojects {
  repositories {
      jcenter()
  }
  group = 'org.jfrog.example.gradle'
  version = '1.1'
  status = 'integration'
}
```

```groovy
publishing {
	publications {
		mavenJava(MavenPublication) {
			from components.java
			pom.withXml {
				asNode().appendNode('description', 'A demonstration of Maven POM customization')
			}
		}
	}
}
```


#### Step 5: Add the defined publication from step '4' and Package information inside the "bintray" closure.

The following package parameters are mandatory:

1. Existing repository in bintray to add the artifacts to (for example: 'generic', 'maven' etc).
2. name for the project
3. userOrg – if not added will use 'BINTRAY_USER' by default
4. License
5. VCS URL


```groovy
bintray {	
    user = 'bintray_user'
    key = 'bintray_api_key'
    publications = ['mavenJava']
    pkg {
		repo = 'generic'
		name = 'gradle-project'
		userOrg = 'bintray_user'
		licenses = ['Apache-2.0']
		vcsUrl = 'https://github.com/bintray/gradle-bintray-plugin.git'
    	}
}
```

#### Step 6: Add a version information inside the "bintray" closure.

In case of defining this, it will override the version stated in the POM xml file and will set the version number in bintray.

```groovy
version {
            name = '1.0-Final' //Bintray logical version name
            desc = //Optional - Version-specific description'
            released  = //Optional - Date of the version release. 2 possible values: date in the format of 'yyyy-MM-dd'T'HH:mm:ss.SSSZZ' OR a java.util.Date instance
            vcsTag = '1.3.0'
            attributes = ['gradle-plugin': 'com.use.less:com.use.less.gradle:gradle-useless-plugin'] //Optional version-level attributes
        }
```

#### Step 7: Run the build

> gradle build bintrayUpload

## The Bintray Plugin DSL
The Gradle Bintray plugin can be configured using its own Convention DSL inside the build.gradle script of your root project.
The syntax of the Convention DSL is described below:

### build.gradle
```groovy
bintray {
    user = 'bintray_user'
    key = 'bintray_api_key'
    
    configurations = ['deployables'] //When uploading configuration files
    // - OR -
    publications = ['mavenStuff'] //When uploading Maven-based publication files
    // - AND/OR -
    filesSpec { //When uploading any arbitrary files ('filesSpec' is a standard Gradle CopySpec)
        from 'arbitrary-files'
        into 'standalone_files/level1'
        rename '(.+)\\.(.+)', '$1-suffix.$2'
    }
    dryRun = false //Whether to run this as dry-run, without deploying
    publish = true //If version should be auto published after an upload
    //Package configuration. The plugin will use the repo and name properties to check if the package already exists. In that case, there's no need to configure the other package properties (like userOrg, desc, etc).
    pkg {
        repo = 'myrepo'
        name = 'mypkg'
        userOrg = 'myorg' //An optional organization name when the repo belongs to one of the user's orgs
        desc = 'what a fantastic package indeed!'
        websiteUrl = 'https://github.com/bintray/gradle-bintray-plugin'
        issueTrackerUrl = 'https://github.com/bintray/gradle-bintray-plugin/issues'
        vcsUrl = 'https://github.com/bintray/gradle-bintray-plugin.git'
        licenses = ['Apache-2.0']
        labels = ['gear', 'gore', 'gorilla']
        publicDownloadNumbers = true
        attributes= ['a': ['ay1', 'ay2'], 'b': ['bee'], c: 'cee'] //Optional package-level attributes
        //Optional version descriptor
        version {
            name = '1.3-Final' //Bintray logical version name
            desc = //Optional - Version-specific description'
            released  = //Optional - Date of the version release. 2 possible values: date in the format of 'yyyy-MM-dd'T'HH:mm:ss.SSSZZ' OR a java.util.Date instance
            vcsTag = '1.3.0'
            attributes = ['gradle-plugin': 'com.use.less:com.use.less.gradle:gradle-useless-plugin'] //Optional version-level attributes
            //Optional configuration for GPG signing
            gpg {
                sign = true //Determines whether to GPG sign the files. The default is false
                passphrase = 'passphrase' //Optional. The passphrase for GPG signing'
            }
            //Optional configuration for Maven Central sync of the version
            mavenCentralSync {
            	sync = true //Optional (true by default). Determines whether to sync the version to Maven Central.
            	user = 'userToken' //OSS user token
            	password = 'paasword' //OSS user password
            	close = '1' //Optional property. By default the staging repository is closed and artifacts are released to Maven Central. You can optionally turn this behaviour off (by puting 0 as value) and release the version manually.
			}            
        }
    }
}
```
* As an example, you can also refer to this multi-module sample project [build file](https://github.com/bintray/bintray-examples/blob/master/gradle-multi-example/build.gradle).

# License
This plugin is available under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

(c) All rights reserved JFrog
