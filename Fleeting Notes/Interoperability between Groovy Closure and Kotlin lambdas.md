#gradle 

Across the Gradle ecosystem there are a lot of instances of `Closure` usage. It becomes a guess work when we are trying to figure out the type for this purpose the Kotlin DSL comes with a [delegateClosureOf](https://gradle.github.io/kotlin-dsl-docs/api/org.gradle.kotlin.dsl/kotlin.-any/delegate-closure-of.html) API.

You can see a usage in action when working with `"org.jfrog.buildinfo:build-info-extractor-gradle:4.28.3"`

```kotlin
import org.gradle.api.Plugin  
import org.gradle.api.Project  
import org.gradle.kotlin.dsl.delegateClosureOf  
import org.gradle.kotlin.dsl.the  
import org.jfrog.gradle.plugin.artifactory.dsl.ArtifactoryPluginConvention  
import org.jfrog.gradle.plugin.artifactory.dsl.PublisherConfig  
import org.jfrog.gradle.plugin.artifactory.dsl.ResolverConfig  
  
class ArtifactoryPlugin : Plugin<Project> {  
    override fun apply(project: Project) = project.run {  
        pluginManager.apply("com.jfrog.artifactory")  
        configureArtifactory()  
    }  
  
    private fun Project.configureArtifactory() {  
        the<ArtifactoryPluginConvention>().run {  
            setContextUrl("https://repo.gradle.org/gradle")  
  
            publish(delegateClosureOf<PublisherConfig> {  
                repository(delegateClosureOf<PublisherConfig.Repository> {})  
            })  
            publish {  
                val config: PublisherConfig = this  
                repository {  
                    val repository: PublisherConfig.Repository = this  
                }  
            }  
            resolve {  
                val resolveConfig: ResolverConfig = this  
            }  
            resolve(delegateClosureOf<ResolverConfig> {  
            })  
        }  
    }  
}
```