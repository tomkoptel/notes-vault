#gradle 
Just use Extension over Convention. [Convention](https://docs.gradle.org/current/javadoc/org/gradle/api/plugins/Convention.html) is planned to be replaced with Extension API eventually.

Extension are plain objects and will be appearing as property on the target object.

>Many Gradle objects are extension aware. This includes; projects, tasks, configurations, dependencies etc.
>
>[Interface ExtensionAware](https://docs.gradle.org/current/javadoc/org/gradle/api/plugins/ExtensionAware.html#)