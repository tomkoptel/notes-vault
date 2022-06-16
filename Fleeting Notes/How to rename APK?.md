#agp_7x 

The process of APK renaming at execution time is a tedious one and requires usage of worker API.

```kotlin
import com.android.build.api.artifact.ArtifactTransformationRequest  
import com.android.build.api.variant.BuiltArtifact  
import java.io.File  
import javax.inject.Inject  
import org.gradle.api.DefaultTask  
import org.gradle.api.file.Directory  
import org.gradle.api.file.DirectoryProperty  
import org.gradle.api.file.RegularFileProperty  
import org.gradle.api.provider.Property  
import org.gradle.api.tasks.Input  
import org.gradle.api.tasks.InputFiles  
import org.gradle.api.tasks.Internal  
import org.gradle.api.tasks.OutputDirectory  
import org.gradle.api.tasks.TaskAction  
import org.gradle.workers.WorkAction  
import org.gradle.workers.WorkParameters  
import org.gradle.workers.WorkerExecutor  
  
abstract class RenameApkArtifactsTask @Inject constructor(  
    private val workers: WorkerExecutor  
) : DefaultTask() {  
    @get:InputFiles  
    abstract val artifactFolder: DirectoryProperty  
  
    @get:OutputDirectory  
    abstract val outFolder: DirectoryProperty  
  
    @get:Input  
    abstract val applicationId: Property<String>  
  
    @get:Internal  
    abstract val transformationRequest: Property<ArtifactTransformationRequest<RenameApkArtifactsTask>>  
  
    @TaskAction  
    fun execute() {  
        transformationRequest.get().submit(  
            this,  
            workers.noIsolation(),  
            WorkItem::class.java  
        ) { builtArtifact: BuiltArtifact, _: Directory, param: WorkItemParameters ->  
            val inputFile = File(builtArtifact.outputFile)  
            val outputFile = inputFile.renameArtifact(  
                versionCode = builtArtifact.versionCode,  
                versionName = builtArtifact.versionName,  
                applicationId = applicationId.get()  
            )  
  
            param.inputApkFile.set(inputFile)  
            param.outputApkFile.set(outputFile)  
            param.outputApkFile.get().asFile  
        }  
    }  
  
    interface WorkItemParameters : WorkParameters, java.io.Serializable {  
        val inputApkFile: RegularFileProperty  
        val outputApkFile: RegularFileProperty  
    }  
  
    abstract class WorkItem @Inject constructor(  
        private val workItemParameters: WorkItemParameters  
    ) : WorkAction<WorkItemParameters> {  
        override fun execute() {  
            val inputFile = workItemParameters.inputApkFile.asFile.get()  
            val outputFile = workItemParameters.outputApkFile.get().asFile  
            // Clean up any existing file  
            outputFile.delete()  
            inputFile.copyTo(outputFile)  
        }  
    }  
}

  
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

Apply to plugin.
```kotlin

override fun apply(project: Project) = project.run {  
    the<ApplicationAndroidComponentsExtension>().run {    
        renameArtifact(project)  
    }
}

private fun Project.renameApkArtifacts(  
    variant: ApplicationVariant,  
) {  
    val suffix = variant.name.capitalize(Locale.getDefault())  
    val task = tasks.register<RenameApkArtifactsTask>("renameApk$suffix") {    
        description = "Renames APK following \$applicationId-v\$versionName(\$versionCode}) for variant $suffix."  
        this.applicationId.set(variant.applicationId)  
    }  
  
    val transformationRequest = variant.artifacts.use(task)  
        .wiredWithDirectories(  
            taskInput = { it.artifactFolder },  
            taskOutput = { it.outFolder },  
        )  
        .toTransformMany(SingleArtifact.APK)  
  
    task.configure {  
        this.transformationRequest.set(transformationRequest)  
    }  
}
```