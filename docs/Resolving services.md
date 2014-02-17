TODO

Method |
----|------|
`object GetInstance(Type pluginType, string instanceKey);` |
`object GetInstance(Type pluginType);` |
`object GetInstance(Type pluginType, Instance instance);` |
`T GetInstance<T>(string instanceKey);` |
`T GetInstance<T>(); `|
`T GetInstance<T>(Instance instance);` |
`IList<T> GetAllInstances<T>();` |
`IList GetAllInstances(Type pluginType);` |
`object TryGetInstance(Type pluginType, string instanceKey);` |
`object TryGetInstance(Type pluginType);` |
`T TryGetInstance<T>();` |
`T TryGetInstance<T>(string instanceKey);` |
`IList<T> GetAllInstances<T>(ExplicitArguments args);` |
`IList GetAllInstances(Type type, ExplicitArguments args);` |
`T GetInstance<T>(ExplicitArguments args);` |
`object GetInstance(Type pluginType, ExplicitArguments args);` | 
`T GetInstance<T>(ExplicitArguments args, string name)` | 

TODO get named instances