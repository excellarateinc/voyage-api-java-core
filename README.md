#voyage-api-java-core

## Project Setup
  - Clone the source from github Repository : https://github.com/lssinc/voyage-api-java-core.git
  - Download Nexus Repository Manager from : https://www.sonatype.com/nexus-repository-oss
  - Unzip the downloaded folder
  - Add the folder path to NEXUS_HOME environment variable and path variable

## Create WAR and upload to Nexus Repository Manager
  - Goto nexus NEXUS_HOME /bin folder and use command "nexus /run"
  - Browse nexus repository using http://localhost:8081/(Default URL and port number) and login with default username and password(admin/admin123).
  - Goto Source folder , Open gradle.properties file and add mavenRepoUsername and mavenRepoPassword. Defaults are admin/admin123
  - Run command from command prompt as "gradle uploadArchives"

## Include WAR in your Project for Usage.
  - Goto gradle.properties file and add WAR path to dependencies (Ex:  `dependencies {compile group: 'io.voyageplatform',                 name: 'voyage-api-core',                version: '1.0.0-RELEASE')}` )

##### Note: If the Jar is already uploaded once, you should remove it from NEXUS Repository to upload it once again.
