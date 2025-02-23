+++
title = "Gradle, the Ultimate Delegator"
outputs = ["Reveal"]

[reveal_hugo]
theme = "simple"
highlight_theme = "idea"
slide_number = true
transition = "slide"
+++

### Gradle the Ultimate Delegator

{{< figure src="images/alligator.jpeg" width=400 height=216 >}}

---

{{% section %}}

### Gradle Delegates to?

{{< figure src="images/delegate-types.png" width=630 height=280 >}}

---

### Project instance

You can access project in any module (e.g. app/build.gradle) also in the root project **build.gradle** file.
The statement is true for Groovy and Kotlin based scripts.

```kotlin
/**
 * getProject() method is useful in build files to
 * explicitly access project properties and methods.
 */
val thisProject: Project = project
```

{{% /section %}}

---

{{% section %}}

### Where it begins? 

```kotlin{}
plugins {
    id("org.jetbrains.kotlin.jvm")
}

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}
```

---

{{< mermaid >}}
timeline
  Startup: bin/java GradleWrapperMain
         : start/connect Daemon
  Configuration: collect scripts
               : select ScriptPlugin
               : compile Program
  Execution: Create delegate PluginDependenciesSpec, Project, Settings, Gradle
           : Evaluate Script
           : Call Program.execute
{{< /mermaid >}}

---

### Partial evaluation

> Interpreter for the Kotlin DSL based on the idea of partial evaluation.
> Instead of interpreting a given Kotlin DSL script directly, the interpreter emits a specialized program.
> Because each program is specialized to a given script structure, a lot of work is avoided.

---

/$GRADLE_HOME/caches/$GRADLE_VERSION/kotlin-dsl/scripts/$CACHE_KEY/instrumented/classes

* Build_gradle.class
* Program.class

---

```kotlin{}
class Build_gradle(
  host: KotlinScriptHost<Project>
) : CompiledKotlinBuildScript by host.project {
  constructor() {
    java {
      sourceCompatibility = JavaVersion.VERSION_21
      targetCompatibility = JavaVersion.VERSION_21
    }
  }
}

class Build_gradle(
  host: KotlinScriptHost<ExtensionAware> 
  pluginDependencies: PluginDependenciesSpec
) : CompiledKotlinPluginsBlock {

  constructor() {
    plugins {
      id("org.jetbrains.kotlin.jvm")
    }     
  }
}
```

{{% /section %}}

---

{{% section %}}

### How Gradle enriches its classpath?

---

{{< slide transition="none" transition-speed="fast" >}}

```kotlin{}
// core/build.gradle.kts
plugins {
    id("org.jetbrains.kotlin.jvm")
}
```

---

/$GRADLE_HOME/caches/$GRADLE_VERSION/kotlin-dsl/scripts/$CACHE_KEY/instrumented/classes

* Build_gradle.class
* Program.class

---

{{< slide transition="none" transition-speed="fast" >}}

```java{}
public final class Program extends ExecutableProgram {
  public void execute(
    ExecutableProgram.Host executableProgramHost, 
    KotlinScriptHost<?> kotlinScriptHost
  ) {
    executableProgramHost.setupEmbeddedKotlinFor(kotlinScriptHost);

    try {
      // The real delegate of the plugins {} block.
      PluginRequestCollector pluginRequestCollector = 
              new PluginRequestCollector(kotlinScriptHost.getScriptSource());
      
      new Build_gradle(kotlinScriptHost, pluginRequestCollector.createSpec(2));
      
      executableProgramHost.applyPluginsTo(
              kotlinScriptHost, 
              pluginRequestCollector.getPluginRequests()
      );
    } catch (Throwable t) {
      executableProgramHost.handleScriptException(t, Build_gradle.class, kotlinScriptHost);
    }

    executableProgramHost.applyBasePluginsTo((Project)kotlinScriptHost.getTarget());
  }
}
```

---

{{< slide transition="none" transition-speed="fast" >}}

```java{9-18}
public final class Program extends ExecutableProgram {
  public void execute(
    ExecutableProgram.Host executableProgramHost, 
    KotlinScriptHost<?> kotlinScriptHost
  ) {
    executableProgramHost.setupEmbeddedKotlinFor(kotlinScriptHost);

    try {
      // The real delegate of the plugins {} block.
      PluginRequestCollector pluginRequestCollector = 
              new PluginRequestCollector(kotlinScriptHost.getScriptSource());
      
      new Build_gradle(kotlinScriptHost, pluginRequestCollector.createSpec(2));
      
      executableProgramHost.applyPluginsTo(
              kotlinScriptHost, 
              pluginRequestCollector.getPluginRequests()
      );
    } catch (Throwable t) {
      executableProgramHost.handleScriptException(t, Build_gradle.class, kotlinScriptHost);
    }

    executableProgramHost.applyBasePluginsTo((Project)kotlinScriptHost.getTarget());
  }
}
```

---

{{< slide transition="none" transition-speed="fast" >}}

```java{12}
public final class Program extends ExecutableProgram {
  public void execute(
    ExecutableProgram.Host executableProgramHost, 
    KotlinScriptHost<?> kotlinScriptHost
  ) {
    executableProgramHost.setupEmbeddedKotlinFor(kotlinScriptHost);

    try {
      PluginRequestCollector pluginRequestCollector = 
              new PluginRequestCollector(kotlinScriptHost.getScriptSource());
      
      new Build_gradle(kotlinScriptHost, pluginRequestCollector.createSpec(2));
      
      executableProgramHost.applyPluginsTo(
              kotlinScriptHost, 
              pluginRequestCollector.getPluginRequests()
      );
    } catch (Throwable t) {
      executableProgramHost.handleScriptException(t, Build_gradle.class, kotlinScriptHost);
    }

    executableProgramHost.applyBasePluginsTo((Project)kotlinScriptHost.getTarget());
  }
}
```

---

{{< slide transition="none" transition-speed="fast" >}}

```java{4-6}
class PluginRequestCollector {
  private final List<PluginDependencySpecImpl> specs = new LinkedList<PluginDependencySpecImpl>();
    
  public PluginDependenciesSpec createSpec(final int pluginsBlockLineNumber) {
    return new PluginDependenciesSpecImpl(pluginsBlockLineNumber);
  }

  private class PluginDependenciesSpecImpl implements PluginDependenciesSpec {
    private final int blockLineNumber;

    public PluginDependenciesSpecImpl(int blockLineNumber) {
      this.blockLineNumber = blockLineNumber;
    }

    @Override
    public PluginDependencySpec id(String id) {
      return id(id, blockLineNumber);
    }

    public PluginDependencySpec id(String id, int requestLineNumber) {
      PluginDependencySpecImpl spec = new PluginDependencySpecImpl(id, requestLineNumber);
      specs.add(spec);
      return spec;
    }
  }
}
```

---

{{< slide transition="none" transition-speed="fast" >}}

```kotlin
class Build_gradle(
    host: org.gradle.kotlin.dsl.support.KotlinScriptHost<*>,
    pluginDependencies: org.gradle.plugin.use.PluginDependenciesSpec
) : org.gradle.kotlin.dsl.support.CompiledKotlinPluginsBlock {
    fun plugins(configure: PluginDependenciesSpec.() -> Unit) {
        configure(pluginDependencies)
    }
}

val build_gradle = Build_gradle(kotlinScriptHost, pluginRequestCollector.createSpec(2))
build_gradle.plugins { 
    id("org.jetbrains.kotlin.jvm")
}
```

---

Calling `plugins {}` dynamically inside the script body, it would not work because
the plugins must be applied at an earlier stage in the build lifecycle.

```kotlin
abstract class KotlinBuildScript(
    private val host: KotlinScriptHost<Project>
) : ProjectDelegate() {
    @Suppress("unused")
    fun plugins(@Suppress("unused_parameter") block: PluginDependenciesSpecScope.() -> Unit): Unit =
        invalidPluginsCall()
}
```

---

```kotlin
plugins {
    id("org.jetbrains.kotlin.jvm")
}

java {
    plugins {
        id("com.android.application")
    }
}
```

---

```
* Exception is:
java.lang.Exception: The plugins {} block must not be used here.
   If you need to apply a plugin imperatively, 
   please use apply<PluginType>() or apply(plugin = "id") instead.
	
	at org.gradle.kotlin.dsl.support.CompiledKotlinBuildScriptKt.invalidPluginsCall(CompiledKotlinBuildScript.kt:143)
	at org.gradle.kotlin.dsl.support.CompiledKotlinBuildScript.plugins(CompiledKotlinBuildScript.kt:61)
	at Build_gradle$2$3.invoke(build.gradle.kts:71)

```

{{% /section %}}

---

{{% section %}}

#### Constructor driven execution

```java
/**
 * class Build_gradle(host: KotlinScriptHost<Project>, $$implicitReceiver0: Project) : CompiledKotlinBuildScript
 */
public final class Build_gradle extends CompiledKotlinBuildScript {
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
public final class Build_gradle extends CompiledKotlinBuildScript {
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

{{% /section %}}

---

{{% section %}}

### Gradle API: Extensions

---

```kotlin
// The same as
// project.extensions.findByType(typeOf<AppExtension>()) 
val myAndroid = project.the<AppExtension>()
```

{{% /section %}}

---

{{% section %}}

### Gradle API: NDOC

---

“A named domain object container is a specialization of NamedDomainObjectSet that adds the ability to create instances of the element type.” (Gradle Docs)

---

{{< figure src="images/thinker.jpg" width=750 height=731 >}}

---

```kotlin{3-5}
android {
  buildTypes {
        release {
            isMinifyEnabled = true
        }
    }
}
```

---

```kotlin{}
val myAndroid = project.the<AppExtension>()

// NamedDomainObjectContainer<BuildType>
val buildTypes = myAndroid.buildTypes

val release: BuildType by buildTypes.getting
val releaseTheSame: BuildType = buildTypes.getByName("release")
val releaseProvider: NamedDomainObjectProvider<BuildType> = 
  buildTypes.named("release") {
    check(release == releaseTheSame)
    check(release == this)
    check(releaseTheSame == this)
  }
```

--- 

### AGP and NDOC
The most known application of NDOC are exposed types under `android` extension.

* productFlavors
* sourceSets
* buildTypes

{{% /section %}}

