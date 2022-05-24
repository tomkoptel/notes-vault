#agp_7x 

# Reference
https://youtu.be/8SFfffaB0CU?t=360

# Code
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