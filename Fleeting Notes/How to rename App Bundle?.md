#agp
First lets define a task. It is pretty straightforward. We do copy one file to another.

```kotlin
import com.android.build.api.variant.BuiltArtifact  
import org.gradle.api.DefaultTask  
import org.gradle.api.file.RegularFileProperty  
import org.gradle.api.provider.Property  
import org.gradle.api.tasks.Input  
import org.gradle.api.tasks.InputFile  
import org.gradle.api.tasks.OutputFile  
import org.gradle.api.tasks.TaskAction  
  
abstract class RenameBundleTask : DefaultTask() {  
    @get:InputFile  
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

To make the magic happening we need to hook our task into `ApplicationVariant` and the override the file output name.

```kotlin
import com.android.build.api.variant.ApplicationAndroidComponentsExtension  
import com.android.build.api.variant.ApplicationVariant
  
private fun ApplicationAndroidComponentsExtension.renameAppBundle(project: Project) {  
    onVariants { variant ->  
        val builtArtifact = variant.artifacts.get(SingleArtifact.APK).map { dir ->  
            variant.artifacts.getBuiltArtifactsLoader().load(dir)  
                ?.elements?.map { it }?.firstOrNull()!!  
        } // 1  
        variant.renameBundleArtifact(project) { bundleFileToRename ->  
            builtArtifact.map {// 4  
                val artifactPrefix = "v${it.versionName}(${it.versionCode})-${variant.name}.aab"  
                File(bundleFileToRename.asFile.parentFile, artifactPrefix)  
            }  
        }    
	}
}  
```

```kotlin
private fun ApplicationVariant.renameBundleArtifact(  
    project: Project,  
    renameArtifact: (RegularFile) -> Provider<File>,  
) {  
    val suffix = name.replaceFirstChar { it.titlecase(Locale.getDefault()) }  
    val task = project.tasks.register<RenameBundleTask>("renameBundle$suffix")  
  
    artifacts.use(task) // 2  
        .wiredWithFiles(  
            taskInput = RenameBundleTask::inArtifact,  
            taskOutput = RenameBundleTask::outArtifact,  
        )  
        .toTransform(SingleArtifact.BUNDLE)  

	// The actual trick is here👇
    task.configure { // 3  
        val bundleFileToRename = outArtifact.get() // 4  
        outArtifact.fileProvider(renameArtifact(bundleFileToRename))) // 5  
    }  
}
```
1. In this particular case we do access the lazy properties of the variant to compute the new name of artifact. It can be replaced with any customisation logic. The most important part is that we have access to the folder where the file will be stored.
2. Here we need to wire new task to the artifacts. As you can see we let the transform action to wire output for us. At this stage it is ok, to let this to be done by AGP.
3. In the next step we do reconfigure the task to override `outArtifact`.
4. It is completely safe to read the `outArtifact` as it was already set and points to some location (e.g. app/build/outputs/bundle/debug/). We are creating new file named in the way we prefer.
5. This line overrides the previously set path to file by AGP to the new file location.