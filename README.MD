<p>
<img src="https://github.com/realm/realm-cocoa/raw/master/logo.png" alt ="realm">
</p>

Realm is a mobile database that runs directly inside phones, tablets or wearables. This repository holds the source code for the Kotlin SDK for Realm, which runs on Kotlin Multiplatform and Android.

## Examples
[https://github.com/realm/realm-kotlin-samples]('https://www.google.ru/')

## Quick Startup

### Prerequisite
Start a new [KMM]('https://www.google.ru/') project. 

### Setup
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
  
_See [Config.kt]('https://www.google.ru/') for the latest version number._
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

### Define model

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

### Open Database
Define a _RealmConfiguration_ with the database schema, then open the Realm using it.
```
val configuration = RealmConfiguration(schema = setOf(Person::class, Dog::class))
```
```
val realm = Realm.open(configuration)
```

### Write
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

### Query
The query language supported by Realm is inspired by Apple’s [NSPredicate]('https://www.google.ru/'), see more examples [here]('https://www.google.ru/')
```
// All Persons
val all = realm.objects<Person>()

// Person named 'Carlo'
val filteredByName = realm.objects<Person>().query("name = $0", "Carlo")

// Person having a dog aged more than 7 with a name starting with 'Fi'
val filteredByDog = realm.objects<Person>().query("dog.age > $0 AND dog.name BEGINSWITH $1", 7, "Fi")
```

### Update
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

### Delete
Use the result of a query to delete from the database
```
// delete all Dogs
realm.beginTransaction()
realm.objects<Dog>().delete()
realm.commitTransaction()
```

Next: head to the full KMM [example]('https://www.google.ru/').
<br>
NOTE: The SDK doesn't currently support `x86` - Please use an `x86_64` or `arm64` emulator/device

## Developer Preview
The Realm Kotlin SDK is in Developer Preview. All API's might change without warning and no guarantees are given about stability. Do not use in production.

### Design documents
The public API of the SDK has not been finalized. Design discussions will happen in both Google Doc and this Github repository. Most bigger features will first undergo a design process that might not involve code. These design documents can be found using the following links:
+ [Intial Project Description]('https://www.google.ru/')
+ [API Design Overview]('https://www.google.ru/')

## How to build locally:
### Prerequisites
+ Swig. On Mac this can be installed using Homebrew: `brew install swig`.
+ CMake 3.18.1. Can be installed through the Android SDK Manager.
### Commands to build from source
```
git submodule update --init --recursive
cd packages
./gradlew assemble
```

