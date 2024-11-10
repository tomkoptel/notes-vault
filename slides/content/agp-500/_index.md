+++
title = "With AGP to 500+ white label apps"
outputs = ["Reveal"]
  
[reveal_hugo]  
theme = "dracula"
highlight_theme = "night-owl"
slide_number = true 
transition = "slide"
+++


### With AGP to 500+ white label apps
{{< figure src="images/grandroid.jpeg" title="A Droid elephant" height=200 width=200 >}}

---
### Gradle 101

* Lifecycle
* Implicit task dependency
* Provider APIs

---
### Lifecycle 3 stages

{{< mermaid >}}
flowchart TD
    A[Initialization Phase] --> B[Configuration Phase]
    B --> C[Execution Phase]

    A -->|Setup build environment| B
    B -->|Configure tasks| C
    C -->|Run tasks| D[Build Complete]
{{< /mermaid >}}

---
{{% section %}}
### Implicit  Task Dependency
---
#### Managed Type

Ultimately, the annotated properties end up inside

`org.gradle.api.tasks.TaskInputs` and 

`org.gradle.api.tasks.TaskOutputs` as "these types have their state entirely managed by Gradle".

---
#### Producer Task

```kotlin{}
abstract class ProducerTask: DefaultTask() {  
    @get:OutputFile  
    abstract val outputFile: RegularFileProperty  
  
    @TaskAction  
    fun action() {  
        logger.lifecycle("Producer: ${outputFile.get()}")  
        outputFile.get().asFile.writeText("foobar")  
    }  
}  
```

---
#### Consumer Task
```kotlin{}
abstract class ConsumerTask : DefaultTask() {  
    @get:InputFile  
    abstract val inputFile: RegularFileProperty  
  
    @TaskAction  
    fun test() {  
        logger.lifecycle("Consumer: ${inputFile.get()}, Content: ${inputFile.get().asFile.readText()}")  
    }  
}
```

---
## Bad Wiring

You should not configure paths manually like that.

```kotlin{4,7}
val myFile = project.layout.buildDirectory.file("license.txt")

project.tasks.register<ProducerTask>(name = "produceFile") {  
    outputFile.set(myFile)  
}  
project.tasks.register<ConsumerTask>(name = "consumeFile") {  
    inputFile.set(myFile)  
}
```

---
## Good Wiring

You should wire task output properties to task input properties.

```kotlin{3,5,8}
val myFile = project.layout.buildDirectory.file("license.txt")

val producerTask: TaskProvider<ProducerTask> =
project.tasks.register<ProducerTask>(name = "produceFile") {  
    outputFile.set(myFile)  
}  
project.tasks.register<ConsumerTask>(name = "consumeFile") {  
    inputFile.set(producerTask.flatMap { it.outputFile })  
}
```

---

An added benefit of connecting input and output properties like this is that **Gradle automatically detects** task dependencies based on such connections.

{{% /section %}}

---
{{% section %}}
### Provider API 101

`Provider` Represents a value that can only be queried and cannot be changed.

---
`Property` extends `Provider`.
`Property` Represents a value that can be queried and changed.

---
`Provider` is a tool for lazy evaluation.

```kotlin{}
Observable.fromCallable {} // RX

callbackFlow {} // coroutine
```

---
**Gradle 9** will apply bytecode transforms [behind the scenes](https://blog.gradle.org/road-to-gradle-9#lazy-apis-and-bytecode-transforms).

{{% /section %}}

---

{{% section %}}
### Setup
---

* settings.gradle.kts
* app/build.gradle.kts
* build-logic/
	* build.gradle.kts
	* settings.gradle.kts
	* android/build.gradle.kts

--- 

### settings.gradle.kts

```kotlin{}
includeBuild("build-logic")
```

---
### app/build.gradle.kts

```kotlin{}
plugins {  
	id("build.logic.android.metadata")  
}
```

---
### [Binary Plugin](https://docs.gradle.org/current/userguide/custom_plugins.html#sec:custom_plugins_standalone_project)

```kotlin{9-19}
plugins {  
	`java-gradle-plugin`  
}

dependencies {
	implementation("com.android.tools.build:gradle:8.6.1")
}

gradlePlugin {  
    plugins {  
        val pluginId = "build.logic.android.metadata"  
        create(pluginId) {  
            id = pluginId  
            implementationClass = "my.package.AndroidMetadataPlugin"  
            version = "1.0"  
            group = "build.logic"  
        }  
    }
}
```

---
### Hooking Into Plugin

```kotlin{6-8}
import org.gradle.api.Plugin  
import org.gradle.api.Project  
  
abstract class AndroidMetadataPlugin : Plugin<Project> {  
    override fun apply(project: Project) {  
        pluginManager.withPlugin("com.android.application") {
           setupPlugin(project)
        }
    }  
}
```

---
### Peek Your Extension

```kotlin{1-3,6-9}
import org.gradle.kotlin.dsl.the
import com.android.build.gradle.AppExtension
import com.android.build.api.variant.ApplicationAndroidComponentsExtension

private fun setupPlugin(project: Project) = project.run {
	// Older API with a lot of tech debt ¯\_(ツ)_/¯
	the<AppExtension>().run {}
	// Latest API available since 2020
	the<ApplicationAndroidComponentsExtension>().run {} 
}
```

---
**ApplicationAndroidComponentsExtension**

* Less coupling to internal impl details
* Better compatibility with [the lazy configuration](https://docs.gradle.org/current/userguide/lazy_configuration.html)
* Older plugin leaked abstractions, lacked clear definition of which API stable/experimental

{{% /section %}}

---

{{% section %}}
### 'New' Variant API
* beforeVariants
* onVariants
---
### Variant?
The combination of build types and product flavor creates variants and test components.

['debug', 'release'] + 'premium' -> `debugPremium`, `releasePremium`

---
**beforeVariants** enable/disable particular component

{{% fragment %}}**beforeVariants** API doesn’t use properties since the values are used at configuration time{{% /fragment %}}

--- 

**onVariants**API is invoked for each variant that was enabled

{{% fragment %}}**onVariants** API makes use of Gradle Properties and Providers{{% /fragment %}}

---
{{< slide transition="none" transition-speed="fast" >}}

### Disable tests for 'release' build type

```kotlin{}
androidComponents {  
    val onRelease = selector().withBuildType("release")  
    beforeVariants(onRelease) { 
	    variantBuilder: ApplicationVariantBuilder ->  
        
        variantBuilder.enableUnitTest = false  
    }  
}
```
---

{{< slide transition="none" transition-speed="fast" >}}

### Disable tests for 'release' build type

```kotlin{6}
androidComponents {  
    val onRelease = selector().withBuildType("release")  
    beforeVariants(onRelease) { 
	    variantBuilder: ApplicationVariantBuilder ->  
        
        variantBuilder.enableUnitTest = false  
    }  
}
```

---

{{< figure src="images/disable-tests.gif" height=305 width=850 >}}

{{% /section %}}

---

{{% section %}}

### 'New' Artifacts API

---

### Implicit task wiring  👎

```kotlin{}
abstract class AarUploadTask : DefaultTask() {
	@get:InputFile
	@get:PathSensitive(PathSensitivity.NONE)
	abstract val aarLocation: RegularFileProperty
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Implicit task wiring  👎

```kotlin{1-3}
the<LibraryExtension>().libraryVariants.configureEach {
	val aarProvider = 
		packageLibraryProvider.flatMap { it.archiveFile } 
		
	project.tasks.register<AarUploadTask>(
		name = "${name}AarUpload"
	) {
		aarLocation.set(aarProvider)
	}
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Implicit task wiring  👎

```kotlin{5-9}
the<LibraryExtension>().libraryVariants.configureEach {
	val aarProvider = 
		packageLibraryProvider.flatMap { it.archiveFile } 
		
	project.tasks.register<AarUploadTask>(
		name = "${name}AarUpload"
	) {
		aarLocation.set(aarProvider)
	}
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Wiring with Artifacts API 👍

```kotlin{1-5}
val ext = the<ApplicationAndroidComponentsExtension>()
ext.onVariants { variant ->
	project.tasks.register<AarUploadTask>(
		name = "${variant.name}AarUpload"
	) {
		aarLocation.set(variant.artifacts.get(SingleArtifact.AAR))
	}
}
```
---
{{< slide transition="none" transition-speed="fast" >}}

### Wiring with Artifacts API 👍

```kotlin{5-8}
val ext = the<ApplicationAndroidComponentsExtension>()
ext.onVariants { variant ->
	project.tasks.register<AarUploadTask>(
		name = "${variant.name}AarUpload"
	) {
		aarLocation.set(variant.artifacts.get(SingleArtifact.AAR))
	}
}
```

---

**No direct dependency** on the internal Android plugin task.

{{% fragment %}}Explicit **cardinality** (the number of elements value holds) [SingleArtifact](https://developer.android.com/reference/tools/gradle-api/8.7/com/android/build/api/artifact/SingleArtifact), [MultipleArtifact](https://developer.android.com/reference/tools/gradle-api/8.7/com/android/build/api/artifact/MultipleArtifact), [ScopedArtifact](https://developer.android.com/reference/tools/gradle-api/8.7/com/android/build/api/artifact/ScopedArtifact).{{% /fragment %}}

---

### SingleArtifact

<style type="text/css">
ul.large-font { font-size: 0.7em; }
</style>

<ul class="large-font">
  <li><b>AAR</b> - The final AAR file as it would be published.</li>
  <li><b>APK</b> - Directory where APK files will be located.</li>
  <li><b>APK_FROM_BUNDLE</b> - Universal APK that contains assets for all screen densities.</li>
  <li><b>ASSETS</b> - Assets that will be packaged in the resulting APK or Bundle.</li>
  <li><b>BUNDLE</b> - The final Bundle ready for consumption at Play Store.</li>
  <li><b>MERGED_MANIFEST</b> - Merged manifest file that will be used in the APK, Bundle and InstantApp packages.</li>
  <li><b>MERGED_NATIVE_LIBS</b> - The directory containing all the native library (.so) files that will be packaged in the APK, AAR, or Bundle.</li>
  <li><b>METADATA_LIBRARY_DEPENDENCIES_REPORT</b> - The metadata for the library dependencies.</li>
  <li><b>PUBLIC_ANDROID_RESOURCES_LIST</b> - A file containing the list of public resources exported by a library project.</li>
  <li><b>RUNTIME_SYMBOL_LIST</b> - The text symbol output file (R.txt) containing a list of resources and their ids (including of transitive dependencies).</li>
</ul>

---

### MultipleArtifact

- **MULTIDEX_KEEP_PROGUARD** - Text files with additional ProGuard rules to be used to determine which classes are compiled into the main dex file.
- **NATIVE_DEBUG_METADATA** - Directories with native debug metadata.
- **NATIVE_SYMBOL_TABLES** - Directories with debug symbol table.


---

### ScopedArtifact

- **CLASSES** - .class files, result of sources compilation and/or external dependencies depending on the scope; includes users' transformation, but does not include Jacoco instrumentation.
- **JAVA_RES** - java resources, result of sources compilation and/or external dependencies depending on the scope.

{{% /section %}}

---	

{{% section %}}

### Renaming APK 🙄

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{1-4}
fun ApplicationAndroidComponentsExtension.renameApk(
  projet: Project,
  loadCommunityNativeConfig: TaskProvider<LoadCommunityNativeConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.forApk(project, outputImpl, loadCommunityNativeConfig).apply {
  	applyTo(output)
  	applyTo(variant)
  }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{5-6}
fun ApplicationAndroidComponentsExtension.renameApk(
  projet: Project,
  loadCommunityNativeConfig: TaskProvider<LoadCommunityNativeConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.forApk(project, outputImpl, loadCommunityNativeConfig).apply {
  	applyTo(output)
  	applyTo(variant)
  }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{8-10}
fun ApplicationAndroidComponentsExtension.renameApk(
  projet: Project,
  loadCommunityNativeConfig: TaskProvider<LoadCommunityNativeConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.forApk(project, outputImpl, loadCommunityNativeConfig).apply {
  	applyTo(output)
  	applyTo(variant)
  }
}
```

{{% /section %}}

--- 
### QA
