# spdx-gradle-plugin
A prototype spdx gradle plugin

⚠ This project is not ready for use to satisfy any real SBOM requirements ⚠

Try it out and see what works, don't depend on it yet, it will probably change

## Usage
This plugin is not published to mavenCentral or gradlePluginPortal, you need to build and deploy
locally and then use in your project

Install into local maven
```bash
$ git clone git@github.com:spdx/spdx-gradle-plugin
$ ./gradlew publishToMavenLocal
```

Add mavenLocal as a plugin repository (settings.gradle.kts)
```kotlin
pluginManagement {
 repositories {
     mavenLocal()
     gradlePluginPortal()
 }
}
```

Apply and configure the plugin
```kotlin
plugins {
  `java`
  ...
  id("org.spdx.sbom") version "0.0.1"
}
...
// there is no default build, you *must* specify a target
spdxSbom {
  targets {
    // create a target named "release",
    // this is used for the task name (spdxSbomForRelease)
    // and output file (release.spdx.json)
    create("release") {
      // configure here
    }
  }
}
```

run sbom generation (use --stacktrace to report bugs)
```bash
./gradlew :spdxSbomForRelease
# or use the aggregate task spdxSbom to run all sbom tasks
# ./gradlew :spdxSbom

output in: build/spdx/release.spdx.json
```

Example output for the plugin run on this project is [example.spdx.json](example.spdx.json)

### Configuration

Tasks can be configured via the extension
```kotlin
spdxSbom {
  targets {
    // create a target named "release",
    // this is used for the task name (spdxSbomForRelease)
    // and output file (release.spdx.json)
    create("release") {
      // use a different configuration (or multiple configurations)
      configurations.set(listOf("myCustomConfiguration"))

      // provide scm info (usually from your CI)
      scm {
        uri.set("my-scm-repository")
        revision.set("asdfasdfasdf...")
      }

      // adjust properties of the document
      document {
        name.set("my spdx document")
        namespace.set("https://my.org/spdx/<some UUID>")
        creator.set("Person:Goose Loosebazooka")

        // add a root spdx package on the document between the document and the
        // root module of the configuration being analyzed, you probably don't need this
        // but it's availalbe if you want to for some reason.
        rootPackage {
          // you must set all or none of these
          name.set("goose")
          version.set("1.2.3")
          supplier.set("Organization:loosebazooka industries")
        }
    }
    // optionally have multiple targets
    // create("another") {
    // }
  }
}
```

### Notes
- Licensing and copyright is somewhat incomplete (works well for maven deps)
- Output is always json

### Experimental (do not use)

If you use these experimental features, they will change them whenever with no notification. They are
to support very specific build usecases and are not for general consumption

use `taskExtension` to map downloadLocations if they are cached somewhere other than original location
```kotlin
tasks.withType<SpdxSbomTask> {
   taskExtension.set(object : SpdxSbomTaskExtension {
       override fun mapRepoUri(input: URI?, moduleId: ModuleVersionIdentifier?): URI {
           // ignore input and return duck
           return URI.create("https://duck.com/repository")
       }
       override fun mapScmForProject(original: ScmInfo?, projectInfo: ProjectInfo?): ScmInfo {
           // ignore provided scminfo (from extension) and project info (the project we are looking for scm info)
           return ScmInfo.from("github.com/goose", "my-sha-is-also-a-goose");
       }
   })
}
```

You can use the abstract class `DefaultSpdxSbomTaskExtension` if you don't want to implement all the methods
of the interface `SpdxSbomTaskExtension`.
