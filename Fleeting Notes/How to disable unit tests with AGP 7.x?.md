#agp_7x 
# Reference
https://youtu.be/LPzBVtwGxlo?t=418
# Code
```kotlin
import com.android.build.api.variant.ApplicationAndroidComponentsExtension  
import com.android.build.api.variant.ApplicationVariantBuilder  
import org.gradle.api.Plugin  
import org.gradle.api.Project  
import org.gradle.kotlin.dsl.the  
  
class CustomizePlaceholderPlugin : Plugin<Project> {  
    override fun apply(project: Project) {  
        project.pluginManager.withPlugin("com.android.application") {  
            project.the<ApplicationAndroidComponentsExtension>().let { extension ->  
                extension.beforeVariants { variantBuilder: ApplicationVariantBuilder ->  
                    if (variantBuilder.name == "debug") {  
                        variantBuilder.unitTestEnabled = false  
                        variantBuilder.minSdk = 23  
                    }  
                }  
            }        
		}    
	}  
}
```