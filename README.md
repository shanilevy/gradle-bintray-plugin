# Overview

The Gradle Bintray Plugin allows you to publish artifacts to Bintray.

[ ![Download](https://api.bintray.com/packages/jfrog/jfrog-jars/gradle-bintray-plugin/images/download.svg) ](https://bintray.com/jfrog/jfrog-jars/gradle-bintray-plugin/_latestVersion)

# Getting Started Using the Plugin
Please follow the below steps to add the Gradle Bintray Plugin to your Gradle build script.

#### Step 1: [Sign up](https://bintray.com/docs/usermanual/working/working_allaboutjoiningbintraysigningupandloggingin.html) and define user and key for [Bintray](https://bintray.com/)

#### Step 2: Apply the plugin to your Gradle build script. 

To apply the plugin, please add one of the following snippets to your `build.gradle` file:

###### Gradle >= 2.1
```groovy
plugins {
    id "com.jfrog.bintray" version "1.2"
}
```
* Currently the "plugins" notation cannot be used for applying the plugin for sub projects, when used from the root build script.

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

#### Step 3: Add the "bintray" closure to your `build.gradle` file.

Add the below "bintray" closure with your bintray user name and key.

```groovy
bintray {
    user = 'bintray_user'
    key = 'bintray_api_key'	
}
```

In case you prefer not to have your Bintray credentials explicitly defined in the script,
you can store them in environment variables and use them as follows:

```groovy
bintray {
    user = System.getenv('BINTRAY_USER')
    key = System.getenv('BINTRAY_KEY')	
}
```


#### Step 4: Add your Bintray package information to the "bintray" closure.

Mandatory parameters:

1. repo - existing repository in bintray to add the artifacts to (for example: 'generic', 'maven' etc).
2. name - package name.
3. licenses - your package licenses.
4. vcsUrl - your VCS URL.

Optional parameters:

1. userOrg – an optional organization name when the repo belongs to one of the user's orgs. If not added will use 'BINTRAY_USER' by default.

```groovy
bintray {	
    user = 'bintray_user'
    key = 'bintray_api_key'
    pkg {
		repo = 'generic'
		name = 'gradle-project'
		userOrg = 'bintray_user'
		licenses = ['Apache-2.0']
		vcsUrl = 'https://github.com/bintray/gradle-bintray-plugin.git'
    }
}
```


#### Step 5: Add a version information to the "pkg" closure.

Mandatory parameters:

1. name - Version name.

Optional parameters:

1. desc - Version description.
2. released - Date of the version release. Can accept one of the following formats: 
	* Date in the format of 'yyyy-MM-dd'T'HH:mm:ss.SSSZZ'
	* java.util.Date instance
3. vcsTag - Version control tag name.
4. attributes - Attributes to be attached to the version.
	

```groovy
pkg {
	version {
	            name = '1.0-Final'
	            desc = 'Gradle Bintray Plugin 1.0 final'
	            released  = new Date()
	            vcsTag = '1.3.0'
	            attributes = ['gradle-plugin': 'com.use.less:com.use.less.gradle:gradle-useless-plugin']
        }
}
```


#### Step 6: Define artifacts to be uploaded to Bintray

Gradle introduces two methods to create groups of artifacts: Publications and Configurations.

Both [Maven Publications](https://docs.gradle.org/current/dsl/org.gradle.api.publish.maven.MavenPublication.html) and [Configurations](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.Configuration.html) can group artifacts to be uploaded to Bintray.
The Maven Publications or Configurations should be added to the Gradle script, outside of the bintray closure.
They should however be referenced from inside the bintray closure.

* Please note that Ivy Publications are not supported.

Here's an example for a Maven Publication that can be added to your Gradle script:

```groovy
publishing {
	publications {
		MyPublication(MavenPublication) {
			from components.java
			groupId 'org.jfrog.gradle.sample'
			artifactId 'gradle-project'
			version '1.1'
		}
	}
}
```

This Publication should be referenced from the bintray closure as follows:

```groovy
bintray {
    user = 'bintray_user'
    key = 'bintray_api_key'	
    publications = ['MyPublication'] 
}
```

As for using Configurations, if for example, you are using the archives Configuration by applying the java plugin

```groovy
apply plugin: 'java'
```
Then the Configuration should be referenced from the bintray closure as follows:

```groovy
bintray {
    user = 'bintray_user'
    key = 'bintray_api_key'	
    configurations = ['archives']
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
