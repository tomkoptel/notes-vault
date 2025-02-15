+++
title = "Gradle, the Ultimate Delegator"
outputs = ["Reveal"]

[reveal_hugo]
theme = "white"
highlight_theme = "idea"
slide_number = true
transition = "slide"
+++

### Gradle, the Ultimate Delegator

{{< figure src="images/alligator.jpeg" title="Graphle" width=400 height=216 >}}

---

{{%section%}}
### Gradle is a Delegator

{{< figure src="images/delegate-types.png" width=630 height=280 >}}

---

### Access Project instance

You can access project in any module (e.g. app/build.gradle) also in the root project **build.gradle** file.

```kotlin
/**
 * Returns this project. This method is useful in build files to
 * explicitly access project properties and methods.
 */
val thisProject: Project = project
```

{{%/section%}}

---
