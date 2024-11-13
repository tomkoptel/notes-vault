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

An added benefit of connecting input and output properties like this is that **Gradle automatically
detects** task dependencies based on such connections.

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
**Gradle 9** will apply bytecode
transforms [behind the scenes](https://blog.gradle.org/road-to-gradle-9#lazy-apis-and-bytecode-transforms).

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
* Better compatibility
  with [the lazy configuration](https://docs.gradle.org/current/userguide/lazy_configuration.html)
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

{{% fragment %}}**beforeVariants** API doesn’t use properties since the values are used at
configuration time{{% /fragment %}}

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

### Implicit task wiring 👎

```kotlin{}
abstract class AarUploadTask : DefaultTask() {
	@get:InputFile
	@get:PathSensitive(PathSensitivity.NONE)
	abstract val aarLocation: RegularFileProperty
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Implicit task wiring 👎

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

### Implicit task wiring 👎

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

{{% fragment %}}Explicit **cardinality** (the number of elements value
holds) [SingleArtifact](https://developer.android.com/reference/tools/gradle-api/8.7/com/android/build/api/artifact/SingleArtifact), [MultipleArtifact](https://developer.android.com/reference/tools/gradle-api/8.7/com/android/build/api/artifact/MultipleArtifact), [ScopedArtifact](https://developer.android.com/reference/tools/gradle-api/8.7/com/android/build/api/artifact/ScopedArtifact).{{%
/fragment %}}

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

- **MULTIDEX_KEEP_PROGUARD** - Text files with additional ProGuard rules to be used to determine
  which classes are compiled into the main dex file.
- **NATIVE_DEBUG_METADATA** - Directories with native debug metadata.
- **NATIVE_SYMBOL_TABLES** - Directories with debug symbol table.

---

### ScopedArtifact

- **CLASSES** - .class files, result of sources compilation and/or external dependencies depending
  on the scope; includes users' transformation, but does not include Jacoco instrumentation.
- **JAVA_RES** - java resources, result of sources compilation and/or external dependencies
  depending on the scope.

{{% /section %}}

---	

{{% section %}}

### Loading remote configuration

---

```kotlin{}
abstract class DevCommunityExtension {
  interface ConfigCredentials {
    @get:Input
    val username: Property<String>
  
    @get:Input
    val password: Property<String>
  
    @get:Input
    val env: Property<Env>
  
    @get:Input
    @get:Optional
    val baseUrl: Property<String>
  }
}
```

---

```kotlin{}
project.extensions.create(
  "devCommunity", 
  DevCommunityExtension::class.java
)
```

---

app/build.gradle

```groovy{}
plugins {
    id("com.android.application")
    id("build.logic.android.metadata")
}

def customCommunity = file("devCommunity.gradle")
if (customCommunity.exists()) {
  apply(from: customCommunity)
}
```

---

app/devCommunity.gradle

```groovy{}
devCommunity {
    name = "androidbudapest"
    configCredentials {
        username = "dev"
        password = "qwerty"
    }
}
```

---

.gitignore

```gitignore{}
devCommunity.gradle
```

---

```kotlin{}
abstract class LoadRemoteConfig : DefaultTask() {
    @get:Nested
    abstract var configCredentials: DevCommunityExtension.ConfigCredentials

    @get:Input
    abstract val community: Property<String>

    @get:OutputFile
    abstract val outArtifact: RegularFileProperty
    
    @TaskAction
    fun execute() { /* ... */ }
}
```

---

```kotlin{}
@TaskAction
fun execute() { /* ... */ }

internal interface NativeConfigApi {
    @GET("/my/api/{communityName}")
    fun loadConfig(
      @Path("communityName") communityName: String
    ): Call<ResponseBody>
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{3-4}
@TaskAction
fun execute() { 
 val api: NativeConfigApi = configCredentials.nativeConfigApi(logger)
 val response = nativeConfigApi.loadConfig(community.get()).execute()
 val source = response.body()?.source()
 val output = outArtifact.get().asFile
 output.parentFile.mkdirs()
 output.sink().buffer().use { sink ->
     sink.writeAll(source)
 }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{5-11}
@TaskAction
fun execute() { 
 val api: NativeConfigApi = configCredentials.nativeConfigApi(logger)
 val response = nativeConfigApi.loadConfig(community.get()).execute()
 val output = outArtifact.get().asFile.apply {
  parentFile.mkdirs()
 }
 output.sink().buffer().use { sink ->
   val source = response.body()?.source()
   sink.writeAll(source)
 }
}
```

---

```kotlin{}
fun Provider<RegularFile>.toNativeConfig(): Provider<NativeConfig> =
  map { it.asFile.toNativeConfig() }

fun File.toNativeConfig(): NativeConfig {
  // load json from file
}

data class NativeConfig(
  val communityId: String,
  /* ... */
)
```

{{% /section %}}

---

{{% section %}}

### Renaming APK, **appId**, **versionCode**, **versionName**

---

Make a network call to get the remote configuration for the app.
{{% fragment %}}Prepare **appId** based on the remote configuration.{{% /fragment %}}
{{% fragment %}}Prepare **versionCode**, **versionName** based on the environment variables.{{%
/fragment %}}
{{% fragment %}}Finally wire providers to Variant/Output types.{{%
/fragment %}}

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{1-4}
fun ApplicationAndroidComponentsExtension.renameApk(
  projet: Project,
  LoadRemoteConfig: TaskProvider<LoadRemoteConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.forApk(project, outputImpl, LoadRemoteConfig).apply {
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
  LoadRemoteConfig: TaskProvider<LoadRemoteConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.forApk(project, outputImpl, LoadRemoteConfig).apply {
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
  LoadRemoteConfig: TaskProvider<LoadRemoteConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.forApk(project, outputImpl, LoadRemoteConfig).apply {
  	applyTo(output)
  	applyTo(variant)
  }
}
```

---

```kotlin{}
fun forApk(
  project: Project,
  output: VariantOutputImpl,
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>,
): OutputProviders {
  return from(
  	project = project,
  	ext = "apk",
  	fullName = output.fullName,
  	LoadRemoteConfig = LoadRemoteConfig
  )
}
```

---

```kotlin{}
internal class OutputProviders(
  val appId: Provider<String>,
  val versionName: Provider<String>,
  val versionCode: Provider<Int>,
  val outputFileName: Provider<String>,
)
```

---

### Version Name

```kotlin{}
fun getVersionName(project: Project): Provider<String> {
 val propProvider = project.providers.gradleProperty("versionName")

 project.providers
  .environmentVariable("VERSION_NAME")
  .orElse(propProvider)
  .orElse("UNSET")
}
```

---

### Version Code

```kotlin{}
fun getVersionCode(project: Project): Provider<String> {
 val propProvider = project.providers.gradleProperty("versionCode")
 
 project.providers
  .environmentVariable("VERSION_CODE")
  .orElse(buildIdProvider)
  .orElse(propProvider)
  .parseIntOrDefault(0)
}
```

---

### From CLI

```bash
./gradlew :app:assemble \
    -Pcommunity=androidbudapest \
    -PversionName=1.0.0 \
    -PversionCode=1
```

---

### On CI

```bash
- name: Build APP
  shell: bash
  env:
    COMMUNITY: ${{needs.env-setup.outputs.community}}
    VERSION_CODE: ${{needs.env-setup.outputs.version_code}}
    VERSION_NAME: ${{needs.env-setup.outputs.version_name}}
  run: |
    ./gradlew :app:assemble
```

---

{{< slide transition="none" transition-speed="fast" >}}

### Application ID

```kotlin{7-8}
private fun from(
  project: Project,
  ext: String,
  fullName: String,
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>,
): OutputProviders {
  val getAppId = loadRemoteConfig.flatMap { task ->
	 task.outArtifact.flatMap { output ->
	   project.provider { 
	     output.asFile.toNativeConfig().applicationId 
     }
	 }
  }
  // ...
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Application ID

```kotlin{9-11}
private fun from(
  project: Project,
  ext: String,
  fullName: String,
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>,
): OutputProviders {
  val getAppId = loadRemoteConfig.flatMap { task ->
	 task.outArtifact.flatMap { output ->
	   project.provider { 
	     output.asFile.toNativeConfig().applicationId 
     }
	 }
  }
  // ...
```

---

### Migration to AGP 8.1.1 🙄

The additional wrapping of the file access file with a provider is necessary because the
AGP Analytics services triggers evaluation of `outArtifact` during the configuration phase
**com.android.build.gradle.internal.profile.AnalyticsService**.
The file is not yet there, so the build fails.

---

### Final Output File Name

```kotlin{3-9}
val getVersionName = getVersionName(project)
val getVersionCode = getVersionCode(project)
val getOutputFileName = getVersionName.flatMap { versionName ->
 getVersionCode.flatMap { versionCode ->
  getAppId.map { applicationId ->
   "$applicationId-v$versionName($versionCode)-$fullName.$ext"
  }
 }
}
return OutputProviders(
    appId = getAppId,
    versionName = getVersionName,
    versionCode = getVersionCode,
    outputFileName = getOutputFileName
)
```

---

### Wiring of providers to VariantOutputImpl

```kotlin{}
fun applyTo(output: VariantOutputImpl) {
    output.versionCode.set(versionCode)
    output.outputFileName.set(outputFileName)
    output.versionName.set(versionName)
}
```

---

### Wiring of providers to ApplicationVariant

```kotlin{1-2}
fun applyTo(variant: ApplicationVariant) {
    variant.applicationId.set(appId)
    variant.buildConfigFields.putAll(
        versionCode.map { versionCode ->
            mapOf(
                "BUILD_NUMBER" to BuildConfigField(
                    type = "String",
                    value = "\"${versionCode}\"",
                    comment = null
                )
            )
        }
    )
}
```

{{% /section %}}

---
{{% section %}}

### BuildConfig, Manifest Placeholder wiring

---

### MapProperty

Part of `org.gradle.api.provider`.

```kotlin{}
package com.android.build.api.variant

import org.gradle.api.provider.MapProperty
import java.io.Serializable

interface Variant : Component, HasAndroidResources {
  val buildConfigFields: MapProperty<String, BuildConfigField<out Serializable>>
  val manifestPlaceholders: MapProperty<String, String>
}
```

---

### Variant#buildConfigFields

```kotlin{3-13}
fun applyTo(variant: ApplicationVariant) {
    variant.applicationId.set(appId)
    variant.buildConfigFields.putAll(
        versionCode.map { versionCode ->
            mapOf(
                "BUILD_NUMBER" to BuildConfigField(
                    type = "String",
                    value = "\"${versionCode}\"",
                    comment = null
                )
            )
        }
    )
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Variant#manifestPlaceholders

```kotlin{4-5}
private fun ApplicationAndroidComponentsExtension.setDeeplinkScheme(
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>
) = onVariants { variant ->
  val placeholders = loadRemoteConfig
      .flatMap { it.outArtifact.toNativeConfig() }
      .map { nativeConfig ->
          mapOf(
              "deepLinkScheme" to nativeConfig.deeplinkScheme,
              "deepLinkHost" to nativeConfig.apiEndpoint
          )
      }
  variant.manifestPlaceholders.putAll(placeholders)
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Variant#manifestPlaceholders

```kotlin{6-10}
private fun ApplicationAndroidComponentsExtension.setDeeplinkScheme(
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>
) = onVariants { variant ->
  val placeholders = loadRemoteConfig
      .flatMap { it.outArtifact.toNativeConfig() }
      .map { nativeConfig ->
          mapOf(
              "deepLinkScheme" to nativeConfig.deeplinkScheme,
              "deepLinkHost" to nativeConfig.apiEndpoint
          )
      }
  variant.manifestPlaceholders.putAll(placeholders)
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Variant#manifestPlaceholders

```kotlin{10-13}
private fun ApplicationAndroidComponentsExtension.setDeeplinkScheme(
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>
) = onVariants { variant ->
  val placeholders = loadRemoteConfig
      .flatMap { it.outArtifact.toNativeConfig() }
      .map { nativeConfig ->
          mapOf(
              "deepLinkScheme" to nativeConfig.deeplinkScheme,
              "deepLinkHost" to nativeConfig.apiEndpoint
          )
      }
  variant.manifestPlaceholders.putAll(placeholders)
}
```

{{% /section %}}


---

{{% section %}}

### Renaming Bundle

---

```kotlin{}
onVariants { variant ->
    val providers = OutputProviders.forBundle(project, variant, loadCommunityNativeConfig)
    RenameBundleTask.register(project, variant, providers)
}

abstract class RenameBundleTask : DefaultTask()
```

---

```kotlin{}
fun forBundle(
    project: Project,
    variant: Variant,
    loadCommunityNativeConfig: TaskProvider<LoadCommunityNativeConfig>,
): OutputProviders = from(
    project = project,
    ext = "aab",
    fullName = variant.name,
    loadCommunityNativeConfig = loadCommunityNativeConfig
)
```

---

```kotlin{}
internal class OutputProviders(
    val appId: Provider<String>,
    val versionName: Provider<String>,
    val versionCode: Provider<Int>,
    val outputFileName: Provider<String>,
)
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{1-7}
abstract class RenameBundleTask : DefaultTask() {
    @get:InputFile
    @get:PathSensitive(PathSensitivity.NONE)
    abstract val inArtifact: RegularFileProperty

    @get:OutputFile
    abstract val outArtifact: RegularFileProperty

    @TaskAction
    fun execute() {
      val inputFile = inArtifact.asFile.get()
      inputFile.copyTo(outArtifact.asFile.get(), overwrite = true)
    }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{9-13}
abstract class RenameBundleTask : DefaultTask() {
    @get:InputFile
    @get:PathSensitive(PathSensitivity.NONE)
    abstract val inArtifact: RegularFileProperty

    @get:OutputFile
    abstract val outArtifact: RegularFileProperty

    @TaskAction
    fun execute() {
      val inputFile = inArtifact.asFile.get()
      inputFile.copyTo(outArtifact.asFile.get(), overwrite = true)
    }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{1-7}
fun register(
    project: Project,
    variant: ApplicationVariant,
    providers: OutputProviders,
): TaskProvider<RenameBundleTask> {
  val taskSuffix = variant.name.replaceFirstChar { it.titlecase(Locale.getDefault()) }
  val renameBundleTask = project.tasks.register<RenameBundleTask>("renameBundle$taskSuffix")
  
  variant.artifacts.use(renameBundleTask)
    .wiredWithFiles(
      RenameBundleTask::inArtifact,
      RenameBundleTask::outArtifact
    )
    .toTransform(SingleArtifact.BUNDLE)
  
  renameBundleTask.configure {
      providers.applyTo(outArtifact, outArtifact::fileProvider)
  }
  
  return renameBundleTask
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{9-14}
fun register(
    project: Project,
    variant: ApplicationVariant,
    providers: OutputProviders,
): TaskProvider<RenameBundleTask> {
  val taskSuffix = variant.name.replaceFirstChar { it.titlecase(Locale.getDefault()) }
  val renameBundleTask = project.tasks.register<RenameBundleTask>("renameBundle$taskSuffix")
  
  variant.artifacts.use(renameBundleTask)
    .wiredWithFiles(
      RenameBundleTask::inArtifact,
      RenameBundleTask::outArtifact
    )
    .toTransform(SingleArtifact.BUNDLE)
  
  renameBundleTask.configure {
    providers.applyTo(
      artifact = outArtifact,
      out = { outArtifact.fileProvider(it) }
    )
  }
  
  return renameBundleTask
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{16-21}
fun register(
    project: Project,
    variant: ApplicationVariant,
    providers: OutputProviders,
): TaskProvider<RenameBundleTask> {
  val taskSuffix = variant.name.replaceFirstChar { it.titlecase(Locale.getDefault()) }
  val renameBundleTask = project.tasks.register<RenameBundleTask>("renameBundle$taskSuffix")
  
  variant.artifacts.use(renameBundleTask)
    .wiredWithFiles(
      RenameBundleTask::inArtifact,
      RenameBundleTask::outArtifact
    )
    .toTransform(SingleArtifact.BUNDLE)
  
  renameBundleTask.configure {
    providers.applyTo(
      artifact = outArtifact,
      out = { outArtifact.fileProvider(it) }
    )
  }
  
  return renameBundleTask
}
```

---

```kotlin{}
class OutputProviders {
  fun applyTo(
    artifact: RegularFileProperty, 
    out: (Provider<File>) -> Unit
  ) {
    val outParent = File(artifact.get().asFile.parent)
    out(outputFileName.map { File(outParent, it) })
  }
}
```

---

### Artifacts API



{{% /section %}}

---

### QA
