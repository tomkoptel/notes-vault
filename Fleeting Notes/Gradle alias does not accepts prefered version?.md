According to `alias` implementation `PluginDependency` should be set with required version. For example in your `libs.toml` file it should be:

```toml
[plugins]  
mavenPublish = { id = "com.vanniktech.gradle-maven-publish-plugin", version = { require = "0.20.0" } }
```

```java
@Override  
public PluginDependencySpec alias(Provider<PluginDependency> notation) {  
    PluginDependency pluginDependency = notation.get();  
    // For now we use the _required version_ when a plugin comes from a catalog  
    return id(pluginDependency.getPluginId()).version(pluginDependency.getVersion().getRequiredVersion());  
}
```

Otherwise Gradle will fail with `plugin version '' is invalid: cannot be null or empty`.