#gradle 
Lets assume we have staging and debug build types. This raises a disambiguaty issue as we don't know which one to peek out of two. For the rescue we have an `attributes` that is an extension for the configurations. The key matching instruments are based on `CompatibilityRule` and `DisambiguationRule`.

`DisambiguationRule` I have too many to peek from.
`CompatibilityRule` I have some, but I don't know which one to peek.

>The role of a compatibility rule is to explain what variants are _compatible_ with what the consumer asked for.
>## [Creating attributes in a build script or plugin](https://docs.gradle.org/current/userguide/variant_attributes.html#creating_attributes_in_a_build_script_or_plugin)
