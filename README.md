# DependencyCheck Central Database Docker

Docker-based self-updating central [OWASP DependencyCheck](https://www.owasp.org/index.php/OWASP_Dependency_Check) Database Server.
This is an out-of-the-box solution for the central Enterprise Setup described [here](https://jeremylong.github.io/DependencyCheck/data/database.html).

## Benefits
- Very fast analysis
- Self-updating, thus always up-to-date CVE data
- No manual central database setup required
- Analysis clients do not require internet access (see below)

Clients do not require internet access in general. Clients that want to use the so called _"Central Analyzer"_ (enabled per default) need HTTP/HTTPS access to connect to [Maven Central Repository](https://search.maven.org/). See [OWASP DependencyCheck documentation](https://jeremylong.github.io/DependencyCheck/data/index.html#Downloading_Additional_Information) for further information.


## Setup

### Central Database Server

In order to start the Database Server simply run
```bash
docker run -p 3306:3306 stefanneuhaus/dependencycheck-central-mysql
```

### Analysis clients

All kinds of analysis clients are supported: Gradle, Maven, Ant, Jenkins, CLI. Apply the following changes to your build file:
- add buildscript dependency for `mysql:mysql-connector-java:5.1.44`
- disable database updates triggered by your project: `autoUpdate = false`
- add database connection parameters: `data { ... }`

Example (Gradle):
```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.owasp:dependency-check-gradle:3.0.2'
        classpath 'mysql:mysql-connector-java:5.1.44'
    }
}

apply plugin: 'org.owasp.dependencycheck'

dependencyCheck {
    autoUpdate = false
    data {
        connectionString = "jdbc:mysql://<DC_HOST>:3307/dependencycheck?useSSL=false"
        driver = "com.mysql.jdbc.Driver"
        username = "dc"
        password = "change-me"
    }
}
```

Start the Dependency Analysis:
```bash
./gradlew dependencyCheckAnalyze
```


## Database updates

Updates of the Database are triggered on the hour. Note that the initial update can take quite some time (~30 min on my machine). In order to get reliable analysis results the initial update must have finished successfully. Subsequent updates are incremental ones and should finish within a couple of seconds.


## Compatibility

Plugin versions used in your project to be analyzed (_client_) usually stay compatible to the DependencyCheck Enterprise Docker Database (_server_) for a long time. All client/server combinations with version >= 1.4.1 should work together.
