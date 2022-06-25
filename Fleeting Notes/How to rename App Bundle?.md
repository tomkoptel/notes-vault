#agp

Renamin bundles is not working well, so we just copy artifact and keep one used in the transformation pipeline.

Here is also a trick of us retrieving `BuiltArtifact` metadata to access version code and version name outside the variant output.

```kotlin
val builtArtifact = variant.artifacts.get(SingleArtifact.APK).map { dir ->  
            variant.artifacts.getBuiltArtifactsLoader().load(dir)  
                ?.elements?.map { it }?.firstOrNull()!!  
        }  
```

**The task**

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
  
    @get:Input  
    abstract val applicationId: Property<String>  
  
    @get:Input  
    abstract val builtArtifact: Property<BuiltArtifact>  
  
    @TaskAction  
    fun execute() {  
        val inputFile = inArtifact.asFile.get()  
        val builtArtifact = builtArtifact.get()  
        val outputFileSameDir = inputFile.renameArtifact(  
            versionCode = builtArtifact.versionCode,  
            versionName = builtArtifact.versionName,  
            applicationId = applicationId.get()  
        )  
  
        // Clean up any existing files  
        outputFileSameDir.delete()  
        outArtifact.asFile.get().delete()  
  
        // We need it because we perform transformation and as a result we need intermediate file  
        inputFile.copyTo(outArtifact.asFile.get())  
  
        inputFile.copyTo(outputFileSameDir)  
    }  
}

import java.io.File  
  
internal fun File.renameArtifact(  
    versionCode: Int?,  
    versionName: String?,  
    applicationId: String,  
): File {  
    val artifactPrefix = "${applicationId}-v${versionName}(${versionCode})"  
    val fileExtension = name.substring(name.lastIndexOf(".") + 1)  
    return File(parentFile, "$artifactPrefix.$fileExtension")  
}
```

The apply to plugin.

```kotlin
private fun Project.renameBundleArtifact(variant: ApplicationVariant) {  
    val suffix = variant.name.capitalize(Locale.getDefault())  
    val task = tasks.register<RenameBundleTask>("renameBundle$suffix") {    
        description = "Renames Bundle following \$applicationId-v\$versionName(\$versionCode}) for variant $suffix."  
  
        val builtArtifact = variant.artifacts.get(SingleArtifact.APK).map { dir ->  
            variant.artifacts.getBuiltArtifactsLoader().load(dir)  
                ?.elements?.map { it }?.firstOrNull()!!  
        }  
        this.builtArtifact.set(builtArtifact)  
        this.applicationId.set(variant.applicationId)  
    }  
    variant.artifacts.use(task)  
        .wiredWithFiles(  
            taskInput = { it.inArtifact },  
            taskOutput = { it.outArtifact },  
        )  
        .toTransform(SingleArtifact.BUNDLE)  
}
```