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

{{< figure src="images/grandroid.jpeg" title="Droidle" height=200 width=200 >}}

---

### Disciple Media

{{< figure src="images/disciple.png" height=517 width=274 >}}

---

### Business Case

{{% fragment %}}As an influencer, I want to have a dedicated **community** of followers.{{% /fragment %}}
{{% fragment %}}The **community** represents a niche group tailored to the influencer's topic.{{% /fragment %}}

---

{{% section %}}
### Project Structure/Plugin Setup
---

* settings.gradle.kts
* app/build.gradle
* **app/devCommunity.gradle**
* **build-logic/**
    * **build.gradle.kts**
    * **settings.gradle.kts**
    * **android/build.gradle.kts**

--- 

### settings.gradle.kts

```kotlin{}
includeBuild("build-logic")
```

---

### app/build.gradle

```kotlin{}
plugins {  
	id("build.logic.android.metadata")  
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### [Binary Plugin](https://docs.gradle.org/current/userguide/custom_plugins.html#sec:custom_plugins_standalone_project)

```kotlin{1-8}
// build-logic/android/build.gradle.kts
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
{{< slide transition="none" transition-speed="fast" >}}

### [Binary Plugin](https://docs.gradle.org/current/userguide/custom_plugins.html#sec:custom_plugins_standalone_project)

```kotlin{10-20}
// build-logic/android/build.gradle.kts
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
{{< slide transition="none" transition-speed="fast" >}}

### Hooking Into Plugin

```kotlin{1-5}
import org.gradle.api.Plugin  
import org.gradle.api.Project  
  
abstract class AndroidMetadataPlugin : Plugin<Project> {  
    override fun apply(project: Project) {  
        project.pluginManager.withPlugin("com.android.application") {
           setupPlugin(project)
        }
    }  
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Hooking Into Plugin

```kotlin{6-8}
import org.gradle.api.Plugin  
import org.gradle.api.Project  
  
abstract class AndroidMetadataPlugin : Plugin<Project> {  
    override fun apply(project: Project) {  
        project.pluginManager.withPlugin("com.android.application") {
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

### Why new API?

{{% fragment %}}Older plugin leaked abstractions and lacked a clear definition of which APIs were stable or experimental.{{% /fragment %}}
{{% fragment %}}Better compatibility with [lazy configuration](https://docs.gradle.org/current/userguide/lazy_configuration.html) based on **Provider**/**Property** types.{{% /fragment %}}

{{% /section %}}

---

{{% section %}}

### Disabling tests for 'release' build type

---

### AGP 'New' Variant API

* beforeVariants
* onVariants

---

### Variant?

{{% fragment %}}The combination of build types and product flavors creates variants and test components.{{% /fragment %}}
{{% fragment %}}['debug', 'release'] + 'premium' -> `debugPremium`, `releasePremium`{{% /fragment %}}

---

### beforeVariants

{{% fragment %}}**beforeVariants** allows enabling or disabling specific components.{{% /fragment %}}
{{% fragment %}}The **beforeVariants** API does not use properties because values are needed during configuration.{{% /fragment %}}

---

### onVariants

{{% fragment %}}The **onVariants** API is called for each enabled variant.{{% /fragment %}}
{{% fragment %}}The **onVariants** API utilizes Gradle Properties and Providers.{{% /fragment %}}

---
{{< slide transition="none" transition-speed="fast" >}}

### Variant Builder

```kotlin{1-4}
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

### Variant Builder

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

### Loading remote configuration

---

### Custom Extension

Enables API call authentication.

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

### Use Action API Over Kotlin Lambdas

```kotlin{}
abstract class DevCommunityExtension {
    @get:Input
    abstract val name: Property<String>
    
    @get:Nested
    abstract val configCredentials: ConfigCredentials

    @Suppress("unused") // Public API
    fun configCredentials(action: Action<ConfigCredentials>) {
        action.execute(configCredentials)
    }
```

---

### Register Extension

```kotlin{}
project.extensions.create(
  "devCommunity", 
  DevCommunityExtension::class.java
)
```

---

### app/build.gradle

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

### app/devCommunity.gradle

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

### .gitignore

```gitignore{}
devCommunity.gradle
```

---

### Definition of LoadRemoteConfig

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

### Retrofit API Service

```kotlin{}
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
 val output = outArtifact.get().asFile
 output.parentFile.mkdirs()
 output.sink().buffer().use { sink ->
     val source = response.body()?.source()
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

### Transform Extension Functions 

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

### Steps

{{% fragment %}}Initiate a network call to retrieve the app's remote configuration.{{% /fragment %}}
{{% fragment %}}Configure **appId** using this remote configuration.{{% /fragment %}}
{{% fragment %}}Set up **versionCode** and **versionName** using environment variables.{{% /fragment %}}
{{% fragment %}}Finally, connect providers to Variant/Output types.{{% /fragment %}}

---
{{< slide transition="none" transition-speed="fast" >}}

### Hooking into onVariants

```kotlin{1-4}
fun ApplicationAndroidComponentsExtension.renameApk(
  projet: Project,
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.for(project, "apk", outputImpl, loadRemoteConfig).apply {
  	applyTo(output)
  	applyTo(variant)
  }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Hooking into onVariants

```kotlin{5-6}
fun ApplicationAndroidComponentsExtension.renameApk(
  projet: Project,
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.forApk(project, outputImpl, loadRemoteConfig).apply {
  	applyTo(output)
  	applyTo(variant)
  }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Hooking into onVariants

```kotlin{8-10}
fun ApplicationAndroidComponentsExtension.renameApk(
  projet: Project,
  LoadRemoteConfig: TaskProvider<LoadRemoteConfig>,
) = onVariants { variant ->
  val outputsImpl = variant.outputs.filterIsInstance<VariantOutputImpl>()
  val outputImpl = outputsImpl.firstOrNull { output -> output.fullName == variant.name }!!
  
  OutputProviders.for(project, "apk", outputImpl, loadRemoteConfig).apply {
  	applyTo(output)
  	applyTo(variant)
  }
}
```

---

### References custom providers for renaming artifacts

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
fun for(
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

```kotlin{9-12}
fun for(
  project: Project,
  ext: String,
  fullName: String,
  loadRemoteConfig: TaskProvider<LoadRemoteConfig>,
): OutputProviders {
  val getAppId = loadRemoteConfig.flatMap { task ->
	 task.outArtifact.flatMap { output ->
	   // @since 8.1.1
	   project.provider { 
	     output.asFile.toNativeConfig().applicationId 
     }
	 }
  }
  // ...
```

---

### Migration to AGP 8.1.1 🙄

Wrapping the file access with a provider is necessary because the
AGP Analytics service evaluates `outArtifact` during the configuration phase.
Since the file does not yet exist, this causes the build to fail.

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

{{% fragment %}}Based on the [MapProperty](https://docs.gradle.org/current/javadoc/org/gradle/api/provider/MapProperty.html).{{% /fragment %}}

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

```kotlin{4-13}
// OutputProviders.kt
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

```kotlin{6-11}
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

```kotlin{12}
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

{{% fragment %}}Hook into the **onVariants**.{{% /fragment %}}
{{% fragment %}}Register the Gradle managed task **RenameBundleTask**.{{% /fragment %}}

---

```kotlin{}
onVariants { variant ->
    val providers = OutputProviders.for(project, "bundle", variant, loadRemoteConfig)
    RenameBundleTask.register(project, variant, providers)
}

abstract class RenameBundleTask : DefaultTask()
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Renaming

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

### Renaming

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

### Register Task

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

### Wire task to the Artifacts API

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
      out = { file: File -> outArtifact.fileProvider(file) }
    )
  }
  
  return renameBundleTask
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Acquire AGP plugin path of the artifact

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
      out = { file: File -> outArtifact.fileProvider(file) }
    )
  }
  
  return renameBundleTask
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Reconfigure rename task

```kotlin{1-7}
internal class OutputProviders(
    val outputFileName: Provider<String>,
){
  fun applyTo(
    artifact: RegularFileProperty, 
    out: (Provider<File>) -> Unit
  ) {
    val outParent = File(artifact.get().asFile.parent)
    out(outputFileName.map { fileName: String -> File(outParent, fileName) })
  }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Get Bundle parent folder path

```kotlin{8}
internal class OutputProviders(
    val outputFileName: Provider<String>,
){
  fun applyTo(
    artifact: RegularFileProperty, 
    out: (Provider<File>) -> Unit
  ) {
    val outParent = File(artifact.get().asFile.parent)
    out(outputFileName.map { fileName: String -> File(outParent, fileName) })
  }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Rename file based on custom naming Provider

```kotlin{9}
internal class OutputProviders(
    val outputFileName: Provider<String>,
){
  fun applyTo(
    artifact: RegularFileProperty, 
    out: (Provider<File>) -> Unit
  ) {
    val outParent = File(artifact.get().asFile.parent)
    out(outputFileName.map { fileName: String -> File(outParent, fileName) })
  }
}
```

{{% /section %}}

---

{{% section %}}

### Modifying AndroidManifest 🛠️

{{% fragment %}}Enable or disable **proxy settings** using third-party tools like Charles Proxy.{{% /fragment %}}
{{% fragment %}}Add or remove **permissions**.{{% /fragment %}}
{{% fragment %}}Add or remove **services**.{{% /fragment %}}

---

### General Approach

{{% fragment %}}Create a task to use the manifest file as an **input file**.{{% /fragment %}}
{{% fragment %}}Connect the task with the Artifacts API via **SingleArtifact.MERGED_MANIFEST**.{{% /fragment %}}
{{% fragment %}}Parse the XML to modify its content.{{% /fragment %}}
{{% fragment %}}Write the modified content to a new **output file**.{{% /fragment %}}

---

### Enable Charles Proxying

{{% fragment %}}Register a task to generate **xml/network_security_config.xml**.{{% /fragment %}}
{{% fragment %}}Link the task to AGP using the **Sources API**.{{% /fragment %}}
{{% fragment %}}Register a task to append the network_security_config to the AndroidManifest.{{% /fragment %}}
{{% fragment %}}Connect the task using the Artifacts API with **SingleArtifact.MERGED_MANIFEST**.{{% /fragment %}}

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{1-4}
private fun ApplicationAndroidComponentsExtension.setHttpProxy(
    project: Project, 
    loadRemoteConfig: TaskProvider<LoadRemoteConfig>
) = onVariants { variant ->
  val tasks = project.tasks
  val genNetworkSecurityConfig = tasks.register<GenNetworkSecurityConfig>(
      name = "create${variant.name.capitalize()}NetworkConfig"
  ) {
      configFile.set(loadRemoteConfig.flatMap { it.outArtifact })
  }
  val appendManifestConfigTask = tasks.register<AppendManifestConfigTask>(
      name = "append${variant.name.capitalize()}ManifestNetworkConfig"
  ) {
      networkConfig.set(genNetworkSecurityConfig.flatMap { it.networkConfig() })
  }
  // ...
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{5-10}
private fun ApplicationAndroidComponentsExtension.setHttpProxy(
    project: Project, 
    loadRemoteConfig: TaskProvider<LoadRemoteConfig>
) = onVariants { variant ->
  val tasks = project.tasks
  val genNetworkSecurityConfig = tasks.register<GenNetworkSecurityConfig>(
      name = "create${variant.name.capitalize()}NetworkConfig"
  ) {
      configFile.set(loadRemoteConfig.flatMap { it.outArtifact })
  }
  val appendManifestConfigTask = tasks.register<AppendManifestConfigTask>(
      name = "append${variant.name.capitalize()}ManifestNetworkConfig"
  ) {
      networkConfig.set(genNetworkSecurityConfig.flatMap { it.networkConfig() })
  }
  // ...
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{11-15}
private fun ApplicationAndroidComponentsExtension.setHttpProxy(
    project: Project, 
    loadRemoteConfig: TaskProvider<LoadRemoteConfig>
) = onVariants { variant ->
  val tasks = project.tasks
  val genNetworkSecurityConfig = tasks.register<GenNetworkSecurityConfig>(
      name = "create${variant.name.capitalize()}NetworkConfig"
  ) {
      configFile.set(loadRemoteConfig.flatMap { it.outArtifact })
  }
  val appendManifestConfigTask = tasks.register<AppendManifestConfigTask>(
      name = "append${variant.name.capitalize()}ManifestNetworkConfig"
  ) {
      networkConfig.set(genNetworkSecurityConfig.flatMap { it.networkConfig() })
  }
  // ...
```

---

```kotlin{}
abstract class GenNetworkSecurityConfig : DefaultTask() {
    @get:InputFile
    @get:PathSensitive(PathSensitivity.NONE)
    abstract val configFile: RegularFileProperty

    @get:OutputDirectory
    abstract val resourcesDir: DirectoryProperty
    
    fun networkConfig(): Provider<RegularFile> {
        return resourcesDir.map { dir -> dir.file("xml/network_security_config.xml") }
    }
```

---

```kotlin{}
abstract class AppendManifestConfigTask : DefaultTask() {
    @get:InputFile
    @get:PathSensitive(PathSensitivity.NONE)
    abstract val networkConfig: RegularFileProperty

    @get:InputFile
    @get:PathSensitive(PathSensitivity.NONE)
    abstract val mergedManifest: RegularFileProperty

    @get:OutputFile
    abstract val updatedManifest: RegularFileProperty
```

---
{{< slide transition="none" transition-speed="fast" >}}

### AppendManifestConfigTask

```kotlin{3-6}
@TaskAction
fun taskAction() {
  val manifest = mergedManifest.asFile.get()
  val manifestXml = DocumentBuilderFactory.newInstance()
      .newDocumentBuilder()
      .parse(manifest)
      
  val manifestTag = manifestXml.getElementsByTagName("application").item(0)
  if (manifestTag.nodeType == Node.ELEMENT_NODE) {
      val element = manifestTag as Element
      if (element.hasAttribute("android:networkSecurityConfig")) {
          element.removeAttribute("android:networkSecurityConfig")
      }
      
      val refName = networkConfig.asFile.get().name.let { fileName ->
        fileName.substring(0, fileName.lastIndexOf('.'))
      }
      element.setAttribute("android:networkSecurityConfig", "@xml/$refName")
  }

  val transformer = TransformerFactory.newInstance().newTransformer()
  val newManifest = updatedManifest.get().asFile
  transformer.transform(DOMSource(manifestXml), StreamResult(newManifest))
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### AppendManifestConfigTask

```kotlin{8-19}
@TaskAction
fun taskAction() {
  val manifest = mergedManifest.asFile.get()
  val manifestXml = DocumentBuilderFactory.newInstance()
      .newDocumentBuilder()
      .parse(manifest)
      
  val manifestTag = manifestXml.getElementsByTagName("application").item(0)
  if (manifestTag.nodeType == Node.ELEMENT_NODE) {
      val element = manifestTag as Element
      if (element.hasAttribute("android:networkSecurityConfig")) {
          element.removeAttribute("android:networkSecurityConfig")
      }
      
      val refName = networkConfig.asFile.get().name.let { fileName ->
        fileName.substring(0, fileName.lastIndexOf('.'))
      }
      element.setAttribute("android:networkSecurityConfig", "@xml/$refName")
  }

  val transformer = TransformerFactory.newInstance().newTransformer()
  val newManifest = updatedManifest.get().asFile
  transformer.transform(DOMSource(manifestXml), StreamResult(newManifest))
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### AppendManifestConfigTask

```kotlin{21-23}
@TaskAction
fun taskAction() {
  val manifest = mergedManifest.asFile.get()
  val manifestXml = DocumentBuilderFactory.newInstance()
      .newDocumentBuilder()
      .parse(manifest)
      
  val manifestTag = manifestXml.getElementsByTagName("application").item(0)
  if (manifestTag.nodeType == Node.ELEMENT_NODE) {
      val element = manifestTag as Element
      if (element.hasAttribute("android:networkSecurityConfig")) {
          element.removeAttribute("android:networkSecurityConfig")
      }
      
      val refName = networkConfig.asFile.get().name.let { fileName ->
        fileName.substring(0, fileName.lastIndexOf('.'))
      }
      element.setAttribute("android:networkSecurityConfig", "@xml/$refName")
  }

  val transformer = TransformerFactory.newInstance().newTransformer()
  val newManifest = updatedManifest.get().asFile
  transformer.transform(DOMSource(manifestXml), StreamResult(newManifest))
}
```

{{% /section %}}

---

{{% section %}}

### Downloading App Icons

---

Retrieve the Cloudinary key.
{{% fragment %}}Construct links following the predefined specifications.{{% /fragment %}}
{{% fragment %}}Generate a **WorkAction** based on the URL.{{% /fragment %}}
{{% fragment %}}Each **WorkAction** establishes an HTTP connection and writes the content to a file.{{% /fragment %}}
{{% fragment %}}The task is connected using the **Sources** API from the **Variant** object.{{% /fragment %}}

---
{{< slide transition="none" transition-speed="fast" >}}

### Register LoadAssetsTask

```kotlin{1-7}
val community = devCommunityExtension.resolveName(project).get()
val communityName = community.replaceFirstChar { it.titlecase(Locale.US) }
val task = tasks.register<LoadAssetsTask>(
  name = "${variant.name}${communityName}CommunityAssets"
) {
    nativeConfigFile.set(loadRemoteConfig.flatMap { it.outArtifact })
}
variant.sources.res?.addGeneratedSourceDirectory(
    task,
    LoadAssetsTask::resourcesDir
)
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Register LoadAssetsTask

```kotlin{8-11}
val community = devCommunityExtension.resolveName(project).get()
val communityName = community.replaceFirstChar { it.titlecase(Locale.US) }
val task = tasks.register<LoadAssetsTask>(
  name = "${variant.name}${communityName}CommunityAssets"
) {
    nativeConfigFile.set(loadRemoteConfig.flatMap { it.outArtifact })
}
variant.sources.res?.addGeneratedSourceDirectory(
    task,
    LoadAssetsTask::resourcesDir
)
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Action

```kotlin{}
with(nativeConfigFile.toNativeConfig().get()) {
  with(Cloudinary(cloudinaryUrl)) {
    loadIconLauncher(assetAppIcon)
    assetNotificationIcon?.let { loadIconNotification(it) }
    assetHeaderLogo?.let { loadHeaderLogo(it) }
    assetLaunchScreen?.let { loadBackgroundSplash(it) }
  }
}
```

--- 

### Load Launcher Icon

```kotlin{}
private fun Cloudinary.loadIconLauncher(publicId: String) 
 = downloadAssets(
  publicId = publicId,
  bundle = mapOf(
      "drawable-hdpi/ic_launcher.png" to {
          add(Resize.fill { width(72).run { height(72) } }.mixInColorSpace())
          extension(Format.png())
      }
 // Other specs
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Query work to queue

```kotlin{1-4}
private fun Cloudinary.downloadAssets(
  publicId: String, 
  bundle: Map<String, Image.Builder.() -> Unit>
) {
  val workQueue = getWorkerExecutor().noIsolation()
  bundle.forEach { (path, imagePresets) ->
   val asset = image {
       publicId(publicId)
       imagePresets(this)
   }
   val downloadUrl = asset.generate()
   workQueue.submit(WorkItem::class.java) {
       outputFile.set(resourcesDir.file(path))
       url.set(downloadUrl)
   }
  }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

### Query work to queue

```kotlin{5}
private fun Cloudinary.downloadAssets(
  publicId: String, 
  bundle: Map<String, Image.Builder.() -> Unit>
) {
  val workQueue = getWorkerExecutor().noIsolation()
  bundle.forEach { (path, imagePresets) ->
   val asset = image {
       publicId(publicId)
       imagePresets(this)
   }
   val downloadUrl = asset.generate()
   workQueue.submit(WorkItem::class.java) {
       outputFile.set(resourcesDir.file(path))
       url.set(downloadUrl)
   }
  }
}
```

---

### WorkerExecutor

```kotlin{9-10}
abstract class LoadAssetsTask : DefaultTask() {
    @get:InputFile
    @get:PathSensitive(PathSensitivity.NONE)
    abstract val nativeConfigFile: RegularFileProperty

    @get:OutputDirectory
    abstract val resourcesDir: DirectoryProperty

    @Inject
    abstract fun getWorkerExecutor(): WorkerExecutor

    @TaskAction
    fun execute() {
      // Do the magic 🪄
    }
}
```

---

### A task within task 🤔?

**Unit of work** that is distributed across available processes.

{{< figure src="images/writing-tasks-5.png" title="Parallel Execution" >}}

---
{{< slide transition="none" transition-speed="fast" >}}

### Query work to queue

```kotlin{6-16}
private fun Cloudinary.downloadAssets(
  publicId: String, 
  bundle: Map<String, Image.Builder.() -> Unit>
) {
  val workQueue = getWorkerExecutor().noIsolation()
  bundle.forEach { (path, imagePresets) ->
   val asset = image {
       publicId(publicId)
       imagePresets(this)
   }
   val downloadUrl = asset.generate()
   workQueue.submit(WorkItem::class.java) {
       outputFile.set(resourcesDir.file(path))
       url.set(downloadUrl)
   }
  }
}
```

---

### WorkItem

```kotlin{}
interface WorkItemParameters : WorkParameters, Serializable {
    val outputFile: RegularFileProperty
    val url: Property<String>
}

abstract class WorkItem @Inject constructor() : WorkAction<WorkItemParameters> {
  override fun execute() {
    val parameters: WorkItemParameters = parameters
    // 1. Create file
    // 2. Make a network call 
    // 3. Write content to the file
  }
}
```

{{% /section %}}

---

{{% section %}}

### Testing

---

### For every test run
{{% fragment %}}Create a test repo.{{% /fragment %}}
{{% fragment %}}Replicate android project setup.{{% /fragment %}}
{{% fragment %}}Apply plugin under test under app/build.gradle file.{{% /fragment %}}
{{% fragment %}}Create **GradleRunner**.{{% /fragment %}}

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{4-6}
@Test
fun `validate app version name can be set with env var`() 
  = withServer { server ->
    server.mockNativeConfig(communityUnderTest)
    projectFixture
        .withDevCommunityConfig(server)
        .withAppPlugin("build.logic.android.metadata")
        .prepareProjectStructure()
        .createRunner()
        .withJaCoCo()
        .withEnvironment(mapOf("VERSION_NAME" to "3.58"))
        .withArguments(":app:generateDebugBuildConfig", "--stacktrace")
        .build()
    projectFixture.appBuildConfig("debug") shouldContain """public static final String VERSION_NAME = "3.58";"""
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{1-18}
import okhttp3.mockwebserver.MockResponse
import okhttp3.mockwebserver.MockWebServer

internal object ShareMetadataTestUtils {
    fun ProjectFixture.withDevCommunityConfig(
      server: MockWebServer
    ): ProjectFixture = withInlinedAppScript(
        """
            devCommunity {
                name = "androidbudapes"
                configCredentials {
                    password = "dev"
                    username = "qwerty"
                    baseUrl = "${server.url("/")}"
                }
            }
        """.trimIndent()
    )

    fun MockWebServer.mockNativeConfig(config: NativeConfigDto) {
        enqueue(MockResponse().setBody(Gson().toJson(config)))
    }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{}
import okhttp3.mockwebserver.MockResponse
import okhttp3.mockwebserver.MockWebServer

internal object ShareMetadataTestUtils {
    fun MockWebServer.mockNativeConfig(config: NativeConfigDto) {
        enqueue(MockResponse().setBody(Gson().toJson(config)))
    }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{7-9}
@Test
fun `validate app version name can be set with env var`() 
  = withServer { server ->
    server.mockNativeConfig(communityUnderTest)
    projectFixture
        .withDevCommunityConfig(server)
        .withAppPlugin("build.logic.android.metadata")
        .prepareProjectStructure()
        .createRunner()
        .withJaCoCo()
        .withEnvironment(mapOf("VERSION_NAME" to "3.58"))
        .withArguments(":app:generateDebugBuildConfig", "--stacktrace")
        .build()
    projectFixture.appBuildConfig("debug") shouldContain """public static final String VERSION_NAME = "3.58";"""
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{10-13}
@Test
fun `validate app version name can be set with env var`() 
  = withServer { server ->
    server.mockNativeConfig(communityUnderTest)
    projectFixture
        .withAppPlugin("build.logic.android.metadata")
        .withDevCommunityConfig(server)
        .prepareProjectStructure()
        .createRunner()
        .withJaCoCo()
        .withEnvironment(mapOf("VERSION_NAME" to "3.58"))
        .withArguments(":app:generateDebugBuildConfig", "--stacktrace")
        .build()
    projectFixture.appBuildConfig("debug") shouldContain """public static final String VERSION_NAME = "3.58";"""
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

```kotlin{14-17}
@Test
fun `validate app version name can be set with env var`() 
  = withServer { server ->
    server.mockNativeConfig(communityUnderTest)
    projectFixture
        .withAppPlugin("build.logic.android.metadata")
        .withDevCommunityConfig(server)
        .prepareProjectStructure()
        .createRunner()
        .withJaCoCo()
        .withEnvironment(mapOf("VERSION_NAME" to "3.58"))
        .withArguments(":app:generateDebugBuildConfig", "--stacktrace")
        .build()
    projectFixture.appBuildConfig("debug").shouldContain(
      """public static final String VERSION_NAME = "3.58";"""
    )
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

Based on **org.junit.rules.TemporaryFolder**.

```kotlin{1-4}
class ProjectFixture : TemporaryFolder(
    /** parentFolder **/
    File(System.getProperty("testEnv.workDir"))!!
)) {
  fun prepareProjectStructure(): ProjectFixture {
      root.resolve("app/src/main").mkdirs()
      root.resolve("settings.gradle.kts").writeText("""include("app", "lib")""")
      root.resolve("local.properties").writeText("""sdk.dir=${System.getenv()["ANDROID_HOME"]}""")
  }
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

The property **testEnv.workDir** is configured during the Gradle setup phase.

```kotlin{1-5}
// build-logic/android/build.gradle.kts
tasks.test {
    val testRuns = layout.buildDirectory.dir("testRuns")
    systemProperty("testEnv.workDir", LazyString(testRuns.map { it.asFile.apply { mkdirs() }.absolutePath }))
}

class LazyString(private val source: Lazy<String>) : Serializable {
  constructor(source: () -> String) : this(lazy(source))
  constructor(source: Provider<String>) : this(source::get)

  override fun toString() = source.value
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

Avoid eager initialization of system property. 

```kotlin{7-12}
// build-logic/android/build.gradle.kts
tasks.test {
    val testRuns = layout.buildDirectory.dir("testRuns")
    systemProperty("testEnv.workDir", LazyString(testRuns.map { it.asFile.apply { mkdirs() }.absolutePath }))
}

class LazyString(private val source: Lazy<String>) : Serializable {
  constructor(source: () -> String) : this(lazy(source))
  constructor(source: Provider<String>) : this(source::get)

  override fun toString() = source.value
}
```

---
{{< slide transition="none" transition-speed="fast" >}}

Create sample android app with lib project.

```kotlin{5-15}
class ProjectFixture : TemporaryFolder(
    /** parentFolder **/
    File(System.getProperty("testEnv.workDir")!!
)) {
  fun prepareProjectStructure(): ProjectFixture {
      root.resolve("settings.gradle.kts").writeText(
        """include("app", "lib")"""
      )
      root.resolve("local.properties").writeText(
        """sdk.dir=${System.getenv()["ANDROID_HOME"]}"""
      )
      // app/src/main/AndroidManifest.xml
      // app/build.gradle
      // lib/src/main/AndroidManifest.xml
      // lib/build.gradle
  }
}
```

---

{{< slide transition="none" transition-speed="fast" >}}

Expose **org.gradle.testkit.runner.GradleRunner**.

```kotlin{5-8}
class ProjectFixture : TemporaryFolder(
    /** parentFolder **/
    File(System.getProperty("testEnv.workDir")!!
)) {
  fun createRunner(): GradleRunner {
      val gradleRunner = GradleRunner.create().withPluginClasspath()
      return gradleRunner.withProjectDir(root)
  }
}
```

---

### withPluginClasspath()

{{% fragment %}}Without this, missing plugin errors will occur.{{% /fragment %}}
{{% fragment %}}Includes dependencies of the compiled plugin in the test.{{% /fragment %}}
{{% fragment %}}Generates the **build/pluginUnderTestMetadata/plugin-under-test-metadata.properties** file.{{% /fragment %}}
{{% fragment %}}Incorporates dependencies listed under the **implementation** configuration within the **dependencies{}** section of the **build.gradle** file.{{% /fragment %}}

---

### withJaCoCo()

There is no built-it support for code coverage in TestKit.
Those tests run in separate JVM and configuration of JaCoCo plugin is not taken into account.
See [Gradle forum](https://discuss.gradle.org/t/gradle-plugins-integration-tests-code-coverage-with-jacoco-plugin/12403) post for more details.

---

Apply Jacoco under **build-logic/android/build.gradle**. 

```kotlin{}
plugins {
  `jacoco`
  id("pl.droidsonroids.jacoco.testkit") version "1.0.12"
}
```

---

Copy contents of generated file **build/testkit/test/testkit-gradle.properties** to the repo under test.

```kotlin{1-5}
fun GradleRunner.withJaCoCo(): GradleRunner {
    javaClass.classLoader.getResourceAsStream("testkit-gradle.properties")
        .toFile(File(projectDir, "gradle.properties"))
    return this
}

fun InputStream.toFile(file: File) {
    use { input ->
        file.outputStream().use { input.copyTo(it) }
    }
}
```

---

Indicates the Jacoco version and the destination for the coverage report.

```properties
org.gradle.jvmargs=\
  "-javaagent\:/$PATH_TO_JAR/org.jacoco.agent-0.8.12-runtime.jar\
  \=destfile\=/Users/you/project/build-logic/android/build/jacoco/test.exec"
```

{{% /section %}}

---

# QA

---

<div class="qr-columns">
    <div class="qr-item">
        <h3>Get in Touch</h3>
        <div id="me" class="qr"></div>
    </div>
    <div class="qr-item">
        <h3>This presentation</h3>
        <div id="thisPresentation" class="qr"></div>
    </div>
</div>

<script>
    new QRCode(document.getElementById("me"), {
        text: "https://bento.me/tomkoptel",
        width: 200,
        height: 200,
    });
    new QRCode(document.getElementById("thisPresentation"), {
        text: "https://tomkoptel.github.io/notes-vault/agp-500/",
        width: 200,
        height: 200,
    });
</script>
