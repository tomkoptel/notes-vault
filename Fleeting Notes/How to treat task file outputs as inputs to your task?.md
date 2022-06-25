∑#gradle 

`project.layout.files()` is a key to this mistery. Behind the scene Gradle looks into the passed type and tries to resolve the passed task provider outputs and add them to the `getSourceFiles()`.  We can avoid having this trouble by inheriting from `SourceTask

>One last thing to note: if you are developing a task that takes collections of source files as inputs, like this example, consider using the built-in [SourceTask](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.SourceTask.html). It will save you having to implement some of the plumbing that we put into `ProcessTemplates`.
>[more-about-tasks](https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:link_output_dir_to_input_files)

 
>[SourceTask](https://docs.gradle.org/current/javadoc/org/gradle/api/tasks/SourceTask.html) is a convenience type for tasks that operate on a set of source files.

```kotlin
val copyTemplates by tasks.registering(Copy::class) {
    into(file(layout.buildDirectory.dir("tmp")))
    from("src/templates")
}

tasks.register<ProcessTemplates>("processTemplates2") {
    // ...
    sources(copyTemplates)
}

class ProcessTemplates {
	@SkipWhenEmpty  
	@InputFiles  
	@PathSensitive(PathSensitivity.NONE)  
	abstract fun getSourceFiles(): ConfigurableFileCollection
	  
	fun mySources(inputTask: TaskProvider<*>) {  
	    getSourceFiles().from(project.layout.files(inputTask));  
	}
}
// ...
```