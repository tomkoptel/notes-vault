#gradle 
When we need to pass a lazy computed value we need to get `Provider<Any>` wrapped property. Calling `project.provider {}` gives us not serializable version that does not include the dependencies of the task we try to wire up. Since Gradle 5.6 release the calls to `map/flatMap` returns "the right" provider. It is guaranteed when using `@Input`.

>`@Input` follows Task dependency.
> [From Gradle properties to AGP APIs (Android Dev Summit '19)](https://youtu.be/OTANozHzgPc?t=493)