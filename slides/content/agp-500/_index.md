+++
title = "With AGP to 500+ white label apps"
outputs = ["Reveal"]
  
[reveal_hugo]  
theme = "moon"
highlight_theme = "monokai"
slide_number = true 
transition = "slide"
+++


# With AGP to 500+ white label apps
{{< figure src="images/grandroid.jpeg" title="A Droid elephant" height=200 width=200 >}}

---
{{% section %}}
# Gradle 101

* Lifecycle
* Implicit task dependency
* Provider APIs

---
# 3 stages


```mermaid
flowchart TD
    A[Initialization Phase .] --> B[Configuration Phase .]
    B --> C[Execution Phase .]

    A -->|Setup build environment .| B
    B -->|Configure tasks .| C
    C -->|Run tasks .| D[Build Complete .]
```

---
#### Producer Task

```kotlin{}
abstract class ProducerTask: DefaultTask() {  
    @get:OutputFile  
    abstract val outputFile: RegularFileProperty  
  
    @TaskAction  
    fun action() {  
        logger.lifecycle("Producer: ${outputFile.get()}")  
        outputFile.get().asFile.writeText("foobar")  
    }  
}  
```

---
#### Consumer Task
```kotlin{}
abstract class ConsumerTask : DefaultTask() {  
    @get:InputFile  
    abstract val inputFile: RegularFileProperty  
  
    @TaskAction  
    fun test() {  
        logger.lifecycle("Consumer: ${inputFile.get()}, Content: ${inputFile.get().asFile.readText()}")  
    }  
}
```

---
## Bad Wiring

```kotlin{4,7}
val myFile = project.layout.buildDirectory.file("license.txt")

project.tasks.register<ProducerTask>(name = "produceFile") {  
    outputFile.set(myFile)  
}  
project.tasks.register<ConsumerTask>(name = "consumeFile") {  
    inputFile.set(myFile)  
}
```

---
## Good Wiring

```kotlin{3,5,8}
val myFile = project.layout.buildDirectory.file("license.txt")

val producerTask: TaskProvider<ProducerTask> =
project.tasks.register<ProducerTask>(name = "produceFile") {  
    outputFile.set(myFile)  
}  
project.tasks.register<ConsumerTask>(name = "consumeFile") {  
    inputFile.set(producerTask.flatMap { it.outputFile })  
}
```

	
{{% /section %}}

---
# QA
