#gradle 
If there is a heavy computation that scales with number of iterations it is worth to design the task as context that receives and processes the input collections and the schedules sub tasks out of each item. [According to the Gradle docs](https://docs.gradle.org/current/userguide/worker_api.html#create_a_custom_task_class), one of such cases is processing of the file collection with mapping of each file to MD5.

The easiest mode to peek is `noIsolation()` the one is fastest, but has the lowest constrainits imposed. For instance, queued tasks can share static class memory scope. If for some reason you need to use different libraries at runtime then it is a time for `classLoaderIsolation()` option.

The `classLoaderIsolation()` requires more complex setup as we need to configure [ConfigurableFileCollection](https://docs.gradle.org/current/javadoc/org/gradle/api/file/ConfigurableFileCollection.html) and then pass it to the task. The configuration of latest is possible with creation of custom "configuration" and then passing it as an input to the task.

```java
abstract public class CreateMD5 extends SourceTask {

    @InputFiles
    abstract public ConfigurableFileCollection getCodecClasspath(); 
```

```groovy
configurations.create('codec') { 
    attributes {
        attribute(Usage.USAGE_ATTRIBUTE, objects.named(Usage, Usage.JAVA_RUNTIME))
    }
    visible = false
    canBeConsumed = false
    canBeResolved = true
}

dependencies {
    codec 'commons-codec:commons-codec:1.10' 
}

tasks.register('md5', CreateMD5) {
    codecClasspath.from(configurations.codec) 
    destinationDirectory = project.layout.buildDirectory.dir('md5')
    source(project.layout.projectDirectory.file('src'))
}

```
