#agp
# AGP Variant Outputs
 The primal focus is to customize specifying custom version name and version code for the build. 
# Reference
https://youtu.be/8SFfffaB0CU?t=191

# Code
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
