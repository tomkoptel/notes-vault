# Gradle 101
* Lifecycle
* Implicit task dependency
* Provider APIs

Before we delve deeper into the discussion, let's clarify the importance of the Gradle lifecycle. There are three main stages: initialization, configuration, and execution. As a plugin developer, your goal is to ensure that we push as much work towards the execution phase as possible. Most of the time, noticeable delays occur during the configuration phase.

## Implicit task dependency

```kotlin
abstract class ProducerTask: DefaultTask() {  
    @get:OutputFile  
    abstract val outputFile: RegularFileProperty  
  
    @TaskAction  
    fun action() {  
        logger.lifecycle("Producer: ${outputFile.get()}")  
        outputFile.get().asFile.writeText("foobar")  
    }  
}  
  
abstract class ConsumerTask : DefaultTask() {  
    @get:InputFile  
    abstract val inputFile: RegularFileProperty  
  
    @TaskAction  
    fun test() {  
        logger.lifecycle("Consumer: ${inputFile.get()}, Content: ${inputFile.get().asFile.readText()}")  
    }  
}
```

**Bad Wiring**
```kotlin
val myFile = project.layout.buildDirectory.file("license.txt")

project.tasks.register<ProducerTask>(name = "produceFile") {  
    outputFile.set(myFile)  
}  
project.tasks.register<ConsumerTask>(name = "produceFile") {  
    inputFile.set(myFile)  
}
```
**Good Wiring**
```kotlin
val myFile = project.layout.buildDirectory.file("license.txt")

val producerTask: TaskProvider<ProducerTask> = project.tasks.register<ProducerTask>(name = "produceFile") {  
    outputFile.set(myFile)  
}  
project.tasks.register<ConsumerTask>(name = "produceFile") {  
    inputFile.set(producerTask.flatMap { it.outputFile })  
}
```

* You should not configure paths manually like that.  
* You should wire task output properties to task input properties.
* By wiring the dependencies, you get the intended ~implicit task dependency~ that you actually should use.
The implicit task dependency wired by Gradle behind the scene. Why? Because we are ## [  
using Gradle managed types](https://docs.gradle.org/current/userguide/properties_providers.html#managed_types)  Ultimately, the annotated properties end up inside 
`org.gradle.api.tasks.TaskInputs`
`org.gradle.api.tasks.TaskOutputs` as "these types have their state entirely managed by Gradle".

>An added benefit of connecting input and output properties like this is that Gradle can automatically detect task dependencies based on such connections. To make this happen at code level, any task  
  details associated with this provider (the one on which {@code flatMap} is being called) are ignored.  
  The new provider will use whatever task details are associated with the return value of the transformation.

## More on Provider API
Provider API is a deferred pipeline of transformations. You can relate the Provider concept to the concept of the cold stream (e.g. `Observable.fromCallable {}` or `callbackFlow {}`). The main idea is to defer the execution of a single unit of work. It is useful to know that the underlying output resource is represented by “heavy computation” or a long-running one like IO. In the world of build systems, it is a common task to take a collection of files, transform each file, and yield a result back, which naturally becomes a “slow” operation that we need to defer as much as possible up to the point of the execution.

Provider API exposes map and flatMap operators that allow us to declare the order of execution. In the context of AGP plugin development, using Provider API becomes a common and promoted approach. The usefulness of Provider API is apparent as [Gradle 9 will enforce](https://blog.gradle.org/road-to-gradle-9#lazy-apis-and-bytecode-transforms) its use at the bytecode level.

As we work with the Gradle tasks APIs we use both [Provider](https://docs.gradle.org/current/javadoc/org/gradle/api/provider/Provider.html) and  [Property](https://docs.gradle.org/current/javadoc/org/gradle/api/provider/Property.html)APIs. The easiest way to remember difference between those two is to compare them to Kotlin `val` and `var` keywords on steroids.
* `Provider` Represents a value that can only be queried and cannot be changed.
* `Property` Represents a value that can be queried and changed.