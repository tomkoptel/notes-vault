#agp_7x 
```kotlin
import com.android.build.api.dsl.ApplicationExtension  
import com.android.build.api.variant.ApplicationAndroidComponentsExtension  
import org.gradle.api.Plugin  
import org.gradle.api.Project  
import org.gradle.kotlin.dsl.the  
  
class CustomizePlaceholderPlugin : Plugin<Project> {  
    override fun apply(project: Project) {  
        project.pluginManager.withPlugin("com.android.application") {  
            project.the<ApplicationAndroidComponentsExtension>().let { extension ->  
                extension.finalizeDsl { ext: ApplicationExtension ->  
                    ext.buildTypes.maybeCreate("debug").let { buildType ->  
                        buildType.manifestPlaceholders["hostName"] = "my.community"  
                        buildType.applicationIdSuffix = ".debug"  
                    }  
                }            }        }    }  
}
```