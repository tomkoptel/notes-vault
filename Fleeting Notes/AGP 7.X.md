# AGP Variant Outputs
 The primal focus is to customize specifying custom version name and version code for the build. 
```kotlin

import org.gradle.api.DefaultTask  
import org.gradle.api.file.RegularFileProperty  
import org.gradle.api.provider.ListProperty  
import org.gradle.api.tasks.Input  
import org.gradle.api.tasks.OutputFile  
import org.gradle.api.tasks.TaskAction  
  
abstract class MyVersionNameTask : DefaultTask() {  
    @get:OutputFile  
    abstract val myVersionName: RegularFileProperty  
  
    @TaskAction  
    fun write() {  
        myVersionName.get().asFile.writeText("1.0")  
    }  
}

import com.android.build.api.variant.ApplicationAndroidComponentsExtension  
import org.gradle.api.Plugin  
import org.gradle.api.Project  
import org.gradle.api.provider.Provider  
import org.gradle.kotlin.dsl.register  
import org.gradle.kotlin.dsl.the  
  
class VersionAndroidPlugin : Plugin<Project> {  
    override fun apply(project: Project) {  
        project.pluginManager.withPlugin("com.android.application") {  
            project.the<ApplicationAndroidComponentsExtension>().let { ext ->  
                val nameTask = project.tasks.register<MyVersionNameTask>("myVersionName") {  
                    // value picked does not matter  
                    myVersionName.set(  
                        project.layout.buildDirectory.file(  
                            "intermediates/my-plugin/version-name.txt"  
                        )  
                    )  
                }  
  
                ext.onVariants { variant ->  
                    for (output in variant.outputs) {  
                        val computedVersionName: Provider<String> = nameTask  
                            .flatMap { it.myVersionName }  
                            .map { it.asFile.readText() }  
                        output.versionName.set(computedVersionName)  
                    }  
                }  
            }        }    }  
}
```

# How to list APKs?


```kotlin
import com.android.build.api.artifact.SingleArtifact  
import com.android.build.api.variant.ApplicationAndroidComponentsExtension  
import org.gradle.api.Plugin  
import org.gradle.api.Project  
import org.gradle.kotlin.dsl.register  
import org.gradle.kotlin.dsl.the  
  
class MetadataAndroidPlugin : Plugin<Project> {  
    override fun apply(target: Project) {  
        target.pluginManager.withPlugin("com.android.application") {  
            target.the<ApplicationAndroidComponentsExtension>().let { ext ->  
                ext.onVariants { variant ->  
                    val artifacts = variant.artifacts.get(SingleArtifact.APK).map { directory ->  
                        variant.artifacts.getBuiltArtifactsLoader().load(directory)  
                            ?.elements?.map { artifact -> artifact.outputFile }  
                            .orEmpty()  
                    }  
                    target.tasks.register<ListApkTask>("${variant.name}ListApkTask") {  
                        group = "Verification"  
                        description = "Print SingleArtifact.APK"  
                        outputFiles.set(artifacts)  
                    }  
                }            }        }    }  
}
```

# How to transform the manifest?
```kotlin
import org.gradle.api.DefaultTask  
import org.gradle.api.file.RegularFileProperty  
import org.gradle.api.tasks.OutputFile  
import org.gradle.api.tasks.TaskAction  
  
abstract class TransformManifestTask : DefaultTask() {  
    @get:OutputFile  
    abstract val manifestInput: RegularFileProperty  
    @get:OutputFile  
    abstract val manifestOutput: RegularFileProperty  
  
    @TaskAction  
    fun write() {  
        val manifest = manifestInput.get().asFile.readText()  
        println(manifest)  
        manifestOutput.get().asFile.writeText(manifest)  
    }  
}

import com.android.build.api.artifact.SingleArtifact  
import com.android.build.api.variant.ApplicationAndroidComponentsExtension  
import org.gradle.api.Plugin  
import org.gradle.api.Project  
import org.gradle.kotlin.dsl.register  
import org.gradle.kotlin.dsl.the  
  
class TransformManifestPlugin : Plugin<Project> {  
    override fun apply(project: Project) {  
        project.pluginManager.withPlugin("com.android.application") {  
            project.the<ApplicationAndroidComponentsExtension>().let { ext ->  
                ext.onVariants { variant ->  
                    val task = project.tasks.register<TransformManifestTask>(  
                        "myIntermediateTransformManifest${variant.name.capitalize()}"  
                    ) {  
                        manifestOutput.set(  
                            project.layout.buildDirectory.file(  
                                "intermediates/my-plugin/transformed-manifest.xml"  
                            )  
                        )  
                    }  
  
                    variant.artifacts.use(task)  
                        .wiredWithFiles(  
                            taskInput = { it.manifestInput },  
                            taskOutput = { it.manifestOutput },  
                        )  
                        .toTransform(SingleArtifact.MERGED_MANIFEST)  
                }  
            }        }    }  
}
```