+++
title = "Gradle, the Ultimate Delegator"
outputs = ["Reveal"]

[reveal_hugo]
theme = "simple"
highlight_theme = "idea"
slide_number = true
transition = "slide"
+++

### Gradle, the Ultimate Delegator

{{< figure src="images/alligator.jpeg" title="Graphle" width=400 height=216 >}}

---

{{% section %}}

### Gradle is a Delegator

{{< figure src="images/delegate-types.png" width=630 height=280 >}}

---

### Access Project instance

You can access project in any module (e.g. app/build.gradle) also in the root project **build.gradle** file.
The statement is true for Groovy and Kotlin based scripts.

```kotlin
/**
 * Returns this project. This method is useful in build files to
 * explicitly access project properties and methods.
 */
val thisProject: Project = project
```

{{% /section %}}

---

{{% section %}}

### Gradle KTS Script Model

---

```kotlin
plugins {
    `kotlin-dsl`
    id("java-gradle-plugin")
}

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}
```

---

#### Everything starts with main

```java
public final class Build_gradle extends KotlinBuildScript {
    public static final void main(String[] args) {
        RunnerKt.runCompiledScript(Build_gradle.class, args);
    }
}
```

---

#### Runner

```kotlin
val evaluator = BasicJvmScriptEvaluator()
val evaluationConfiguration: ScriptEvaluationConfiguration = //...
runBlocking {
    evaluator(script, evaluationConfiguration).onFailure {
        it.reports.forEach(System.err::println)
    }
}    

```

---

#### Constructor driven logic

```java
public final class Build_gradle extends KotlinBuildScript {
    public Build_gradle(KotlinScriptHost host) {
        // ...
        Accessors96b3ii45gitqpy1kb3tvcvtxvKt.java((Project)this, Build_gradle::_init_$lambda$0);
    }
}
```

---

#### Finally configure over JavaPluginExtension

```java
private static final void _init_$lambda$0(JavaPluginExtension extension) {
  extension.setSourceCompatibility(JavaVersion.VERSION_21);
  extension.setTargetCompatibility(JavaVersion.VERSION_21);
}
```

---

### What we learned?

Gradle uses the Kotlin compiler’s scripting engine. 
The generated class (commonly named Build_gradle) is the concrete implementation of your build script.

---

### More about Build_gradle

{{% fragment %}}build.gradle.kts is compiled to a class (e.g., Build_gradle).{{% /fragment %}}
{{% fragment %}}Build_gradle extends KotlinBuildScript, which is tailored for Kotlin DSL support.{{% /fragment %}}
{{% fragment %}}KotlinBuildScript uses ProjectDelegate to delegate calls to the actual Project (from org.gradle.api.Project).{{% /fragment %}}
{{% fragment %}}This design gives you a type-safe DSL that directly maps to Gradle's build system.{{% /fragment %}}

{{% /section %}}

---
