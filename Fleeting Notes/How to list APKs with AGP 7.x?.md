#agp

# Reference
https://youtu.be/8SFfffaB0CU?t=300

# Code

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