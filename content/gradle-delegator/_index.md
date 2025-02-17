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
    id("org.jetbrains.kotlin.jvm")
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

#### Constructor driven execution

```java
public final class Build_gradle extends KotlinBuildScript {
    public Build_gradle(KotlinScriptHost host) {
        // ...
        Accessors96b3ii45gitqpy1kb3tvcvtxvKt.java(
                (Project)this,
                Build_gradle::_init_$lambda$0
        );
    }

    private static final void _init_$lambda$0(JavaPluginExtension extension) {
        extension.setSourceCompatibility(JavaVersion.VERSION_21);
        extension.setTargetCompatibility(JavaVersion.VERSION_21);
    }
}
```

---

#### Everything is still Gradle Public API

```java
public final class Build_gradle extends KotlinBuildScript {
    public Build_gradle(KotlinScriptHost host) {
        ExtensionAware extensionAware = (ExtensionAware) this;
        project.getExtensions().configure(
                JavaPluginExtension.class,
                new Action<JavaPluginExtension>() {
            @Override
            public void execute(JavaPluginExtension extension) {
                extension.setSourceCompatibility(JavaVersion.VERSION_21);
                extension.setTargetCompatibility(JavaVersion.VERSION_21);
            }
        });
    }
}
```

---

#### Kotlin equivalent

```kotlin
plugins {
    id("org.jetbrains.kotlin.jvm")
}

// !!! 'this' won't typecast to ExtensionAware
val extensionAware: ExtensionAware = project 
extensionAware.extensions.configure("java", Action<JavaPluginExtension>{
    val extension: JavaPluginExtension = this
    extension.sourceCompatibility = JavaVersion.VERSION_21
    extension.targetCompatibility = JavaVersion.VERSION_21
})
```

---

#### [Or simply use Kotlin DSL](https://github.com/runningcode/kotlin-dsl/blob/master/subprojects/provider/src/main/kotlin/org/gradle/kotlin/dsl/ExtensionAwareExtensions.kt#L41-L49)

```kotlin
plugins {
    id("org.jetbrains.kotlin.jvm")
}

configure<JavaPluginExtension> {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}
```

---

### What we've learned?

Gradle uses the Kotlin compiler’s scripting engine. 
The generated class Build_gradle is the concrete implementation of your build script.

```kotlin
abstract class KotlinBuildScript(
    private val host: KotlinScriptHost<Project>
) : Project by host.target {}
```

{{% /section %}}

---

{{% section %}}

### Gradle Groovy Script Model

---

```groovy
// Simulate Gradle's behavior:
// 1. Create a Project instance.
def projectInstance = new Project("MyAwesomeProject")
// 2. Inject the Project into the Binding.
def binding = new Binding([project: projectInstance])
// 3. Create the build script with the binding.
def scriptInstance = new build_cfj79vrwubnnl9mpc4lxadrlm(binding)
// 4. Run the script.
scriptInstance.run()
```

---

### Script Example

```groovy
Project currentProject = getProject()
def build_gradle = this
```

---

### Script Compiled To

```java
import org.codehaus.groovy.runtime.ScriptBytecodeAdapter;
import org.codehaus.groovy.runtime.callsite.CallSite;

public class build_cfj79vrwubnnl9mpc4lxadrlm 
        extends ProjectScript implements ScriptOrigin {
    public Object run() {
        CallSite[] var1 = $getCallSiteArray();
        Project currentProject = (Project)ScriptBytecodeAdapter.castToType(
                var1[0].callCurrent(this), 
                Project.class
        );
        Object build_gradle = this;
    }
}
```

---

### What we've learned?

Method calls are transformed into **CallSite** objects.
These objects are responsible for resolving and invoking the methods at runtime.
The calls will be delegated to the Project instance.

---
{{% /section %}}

