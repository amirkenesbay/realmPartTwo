<p>
<img src="https://github.com/realm/realm-cocoa/raw/master/logo.png" alt ="realm">
</p>

Realm is a mobile database that runs directly inside phones, tablets or wearables. This repository holds the source code for the Kotlin SDK for Realm, which runs on Kotlin Multiplatform and Android.

# Examples

https://github.com/realm/realm-kotlin-samples

# Quick Startup

## Prerequisite

Start a new [KMM](google.ru) project. 

## Setup
+ Add the following Gradle configuration in the root project (make sure you're using Kotlin `1.4.20` or recent) `<root project>/build.gradle.kts`

```
buildscript {
    repositories {
        // other repo
        maven(url = "https://oss.jfrog.org/artifactory/oss-snapshot-local")
    }
    dependencies {
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:1.4.20")// minimum 1.4.20
        classpath("com.android.tools.build:gradle:4.0.1")
        classpath("io.realm.kotlin:gradle-plugin:0.0.1-SNAPSHOT")
    }
}

allprojects {
    repositories {
        // other repo 
        maven(url = "https://oss.jfrog.org/artifactory/oss-snapshot-local")
    }
}
```
<p>
<img src="https://github.com/realm/realm-kotlin/raw/master/images/RootGradle.png" alt ="realm">
</p>

+ Apply the `realm-kotlin` plugin and specify the dependency in the common source set.
  
_See [Config.kt](https://www.google.ru/) for the latest version number._
```
plugins {
    kotlin("multiplatform")
    id("com.android.library")
    id("realm-kotlin")
}

kotlin {
  sourceSets {
      val commonMain by getting {
          dependencies {
              implementation("io.realm.kotlin:library:0.0.1-SNAPSHOT")
          }
      }
}
```
<p>
<img src="https://github.com/realm/realm-kotlin/raw/master/images/SharedGradle.png" alt="realm">
</p>

## Define model

Start writing your shared database logic in the shared module by defining first your model

```
class Person : RealmObject {
    var name: String = "Foo"
    var dog: Dog? = null
}

class Dog : RealmObject {
    var name: String = ""
    var age: Int = 0
}
```

## Open Database
Define a _RealmConfiguration_ with the database schema, then open the Realm using it.
```
val configuration = RealmConfiguration(schema = setOf(Person::class, Dog::class))
```
```
val realm = Realm.open(configuration)
```

## Write
Persist some data by instantiating the data objects and copying it into the open Realm instance
```
// plain old kotlin object
val person = Person().apply {
    name = "Carlo"
    dog = Dog().apply { name = "Fido"; age = 16 }
}

// persist it in a transaction
realm.beginTransaction()
val managedPerson = realm.copyToRealm(person)
realm.commitTransaction()

// alternatively we can use
realm.beginTransaction()
realm.create<Person>().apply {
            name = "Bar"
            dog = Dog().apply { name = "Filo"; age = 11 }
        }
realm.commitTransaction()
```

## Query
The query language supported by Realm is inspired by Apple’s [NSPredicate](https://www.google.ru/), see more examples [here](https://www.google.ru/)
```
// All Persons
val all = realm.objects<Person>()

// Person named 'Carlo'
val filteredByName = realm.objects<Person>().query("name = $0", "Carlo")

// Person having a dog aged more than 7 with a name starting with 'Fi'
val filteredByDog = realm.objects<Person>().query("dog.age > $0 AND dog.name BEGINSWITH $1", 7, "Fi")
```

## Update
```
// Find the first Person without a dog
realm.objects<Person>().query("dog == NULL LIMIT(1)")
    .firstOrNull()
    ?.also { personWithoutDog ->
        // Add a dog in a transaction
        realm.beginTransaction()
        personWithoutDog.dog = Dog().apply { name = "Laika";  age = 3 }
        realm.commitTransaction()
    }
```

## Delete
Use the result of a query to delete from the database
```
// delete all Dogs
realm.beginTransaction()
realm.objects<Dog>().delete()
realm.commitTransaction()
```

Next: head to the full KMM [example](https://www.google.ru/).
<br>
NOTE: The SDK doesn't currently support `x86` - Please use an `x86_64` or `arm64` emulator/device

# Developer Preview
The Realm Kotlin SDK is in Developer Preview. All API's might change without warning and no guarantees are given about stability. Do not use in production.

## Design documents
The public API of the SDK has not been finalized. Design discussions will happen in both Google Doc and this Github repository. Most bigger features will first undergo a design process that might not involve code. These design documents can be found using the following links:
+ [Intial Project Description](https://www.google.ru/)
+ [API Design Overview](https://www.google.ru/)

# How to build locally:
## Prerequisites
+ Swig. On Mac this can be installed using Homebrew: `brew install swig`.
+ CMake 3.18.1. Can be installed through the Android SDK Manager.
## Commands to build from source
```
git submodule update --init --recursive
cd packages
./gradlew assemble
```

In Android Studio open the `test` project, which will open also the `realm-library` and the compiler projects
<br><br>
You can also run tests from the commandline:
```
cd test
./gradlew connectedAndroidTest
./gradlew macosTest
```

# Using Snapshots
If you want to test recent bugfixes or features that have not been packaged in an official release yet, you can use a -**SNAPSHOT** release of the current development version of Realm via Gradle, available on [JFrog OSS](https://www.google.ru/)
```
// Global build.gradle
buildscript {
    repositories {
        google()
        jcenter()
        maven {
            url 'http://oss.jfrog.org/artifactory/oss-snapshot-local'
        }
        maven {
            url 'https://dl.bintray.com/kotlin/kotlin-dev'
        }
    }
    dependencies {
        classpath 'io.realm.kotlin:plugin-gradle:<VERSION>'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        maven {
            url 'http://oss.jfrog.org/artifactory/oss-snapshot-local'
        }
        maven {
            url 'https://dl.bintray.com/kotlin/kotlin-dev'
        }
    }
}
```

See [Config.kt](https://www.google.ru/) for the latest version number.

# Repository Guidelines

## Branch Strategy

We have three branches for shared development: `master`, `releases` and `next-major`. Tagged releases are only made from `releases`.
<br>
`master`:
+ Target branch for new features.
+ Cotains the latest publishable state of the SDK.
+ [SNAPSHOT releases](https://www.google.ru/) are being created for every commit.
  
`releases`:
+ All tagged releases are made from this branch.
+ Target branch for bug fixes.
+ Every commit should be merged back to master `master`.
+ Minor changes (e.g. to documentation, tests, and the build system) may not affect end users but should still be merged to `releases` to avoid diverging too far from `master` and to reduce the likelihood of merge conflicts.

`next-major`:
+ Target branch for breaking changes that would result in a major version bump.

Note: We currently only have the `master` branch, as no tagged releases have been made yet.

## Code Style

We use the offical [style guide](https://www.google.ru/) from Kotlin which is enforced using [ktlint](https://www.google.ru/) and [detekt](https://www.google.ru/).

```
# Call from root folder to check if code is compliant.
./gradlew ktlintCheck
./gradlew detekt

# Call from root folder to automatically format all Kotlin code according to the code style rules.
./gradlew ktlintFormat
```

Note: ktlint does not allow group imports using `.*`. You can configure IntelliJ to disallow this by going to preferences `Editor > Code Style > Kotlin > Imports` and select "Use single name imports".
## Writing Tests
Currently all unit tests should be place in the `test/` project instead of `packages/library`. The reason for this is that we need to apply the Realm Compiler Plugin to the tests and this introduces a circular dependency if the tests are in `library`.
<br><br>
Inside `tests/` there are 3 locations the files can be placed in:
+ `test/src/commonTest`
+ `test/src/androidTest`
+ `test/src/androidTest`

Ideally all shared tests should be in `commonTest` with specific platform tests in `androidTest/macosTest`. However IntelliJ does not yet allow you run you to run common tests on Android from within the IDE]([https://youtrack.jetbrains.com/issue/KT-46452](https://www.google.ru/)), so we are using the following work-around:
1. All "common" tests should be placed in the `test/src/androidtest/kotlin/io/realm/shared folder`. They should be written using only common API's. I'e. use Kotlin Test, not JUnit. This `io.realm.shared` package should only contain tests we plan to eventually move to `commontTest`.
2. When adding a new test file to `androidTest` we need to re-create the symlinks for macOS. This can be done, using the following command on Mac:
```
cd test/src/macosTest/kotlin/io/realm/shared
ln -sf ../../../../../androidTest/kotlin/io/realm/shared/* ./
```
3. Both the real test file and the symlink must be committed to Git.
4. This allows us to run and debug unit tests on both macOS and Android. It is easier getting the imports correctly using the macOS sourceset as the Android code will default to using JUnit.

All platform specific tests should be placed outside the `io.realm.shared package`, the default being `io.realm`.

## Defining dependencies
All dependency versions and other constants we might want to share between projects are defined inside the file `buildSrc/src/main/kotlin/Config.kt`. Any new dependencies should be added to this file as well, so we only have one location for these.

## Contributing Enhancements
We love contributions to Realm! If you'd like to contribute code, documentation, or any other improvements, please [file a Pull Request](https://www.google.ru/) on our GitHub repository. Make sure to accept our [CLA](https://www.google.ru/)!
<br><br>
This project adheres to the [Contributor Covenant Code of Conduct](https://www.google.ru/). By participating, you are expected to uphold this code. Please report unacceptable behavior to [info@realm.io](https://www.google.ru/).

## CL
Realm welcomes all contributions! The only requirement we have is that, like many other projects, we need to have a [Contributor License Agreement](https://www.google.ru/) (CLA) in place before we can accept any external code. Our own CLA is a modified version of the Apache Software Foundation’s CLA.
<br><br>
[Please submit your CLA electronically using our Google form](https://www.google.ru/) so we can accept your submissions. The GitHub username you file there will need to match that of your Pull Requests. If you have any questions or cannot file the CLA electronically, you can email [help@realm.io](https://www.google.ru/).

# Samples

## Kotlin Multiplatform Sample

The folder `examples/kmm-sample` contains an example showing how to use Realm in a multiplatform project, sharing code for using Realm in the shared module. The project is based on `https://github.com/Kotlin/kmm-sample`.
