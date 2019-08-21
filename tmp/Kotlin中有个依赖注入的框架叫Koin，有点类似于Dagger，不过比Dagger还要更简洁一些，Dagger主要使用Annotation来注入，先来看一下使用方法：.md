Kotlin中有个依赖注入的框架叫Koin，有点类似于Dagger，不过比Dagger还要更简洁一些，Dagger主要使用Annotation来注入，先来看一下使用方法：

1. 定义之后可能使用到的Module，之后在Application回调`onCreate`方法的时候加载这些Module。

```kotlin
private val testModule: Module = module {
    factory { Date(get()) }
    single { ContactManager() }
    viewModel<CallingViewModel>()
}
private val moduleList = listOf(testModule)

override fun onCreate() {
    super.onCreate()
    startKoin(this, moduleList)
}
```

2. 使用的时候通过`by inject`或者`by viewModel`来注入即可

```ko
private val contactManager: ContactManager by inject()
contactManager.search()

private val callingViewModel: CallingViewModel by viewModel()
callingViewModel.dial()
```

这里有三种定义Module的方式

- Single: 单例，如果已经创建过则不再创建
- Factory: 每inject一次就会重新创建一次
- viewModel: 用于Android的MVVM

基本的使用还是挺简单的，接下来看下具体怎么实现的。

```ko
private val testModule: Module = module {
    factory { Date() }
    single { ContactManager() }
    viewModel<CallingViewModel>()
}
```

从源码来看`Module`是一个funtion，这个function的返回值是`ModuleDefinition`，这就是koin的module。

```kot
typealias Module = (KoinContext) -> ModuleDefinition
```

之后将这些function整合成一个list中。也就是`List<(KoinContext) -> ModuleDefinition>`

```kot
private val moduleList = listOf(testModule)
```



然后看下`startKoin`方法做了什么，将传入的list转化成数组，创建Koin实例（单例）。

```kot
fun ComponentCallbacks.startKoin(
    androidContext: Context,
    modules: List<Module>,
    extraProperties: Map<String, Any> = HashMap(),
    loadPropertiesFromFile: Boolean = false,
    logger: Logger = AndroidLogger()
) {
	.......
	
    val koin: Koin = loadKoinModules(modules).with(androidContext)
	
	.......
}


fun loadKoinModules(modules: List<Module>): Koin = loadKoinModules(*modules.toTypedArray())


fun loadKoinModules(vararg modules: Module): Koin {
    return getOrCreateMinimalKoin(modules.toList())
}

private fun getOrCreateMinimalKoin(
        list: List<Module>
    ): Koin = synchronized(this) {
    	// 单例
        if (koin == null) {
            koin = Koin.create()
        }
        koin?.apply {
            loadModules(list)
        }
        return getKoin()
    }
```

调用这个实例的`loadModules`，这个实例中从传入的集合中挨个取出`ModuleDefination`进行注册。

```ko
fun loadModules(modules: Collection<Module>): Koin {
    val duration = measureDuration {
        modules.forEach { module ->
            registerDefinitions(module(koinContext))
        }
    }
    return this
}
```

重点在于这个`registerDefinitions`

```ko
    private fun registerDefinitions(
        moduleDefinition: ModuleDefinition,
        parentModuleDefinition: ModuleDefinition? = null,
        path: Path = Path.root()
    ) {
        val modulePath: Path =
            pathRegistry.makePath(moduleDefinition.path, parentModuleDefinition?.path)
        val consolidatedPath =
            if (path != Path.root()) modulePath.copy(parent = path) else modulePath

        pathRegistry.savePath(consolidatedPath)

        // Add definitions & propagate eager/override
        moduleDefinition.definitions.forEach { definition ->
            val eager =
                if (moduleDefinition.createOnStart) moduleDefinition.createOnStart else definition.isEager
            val override =
                if (moduleDefinition.override) moduleDefinition.override else definition.allowOverride
            val name = if (definition.name.isEmpty()) {
                val pathString =
                    if (consolidatedPath == Path.Companion.root()) "" else "$consolidatedPath."
                "$pathString${definition.primaryTypeName}"
            } else definition.name
            val def = definition.copy(
                name = name,
                isEager = eager,
                allowOverride = override,
                path = consolidatedPath
            )
            // 防止重复注册，先把已经实例出来的module删除。
            instanceFactory.delete(def)
            // 将ModuleDefinition下的BeanDefinition加入到beanRegistry中的HashSet中
            beanRegistry.declare(def)
        }

        // Check sub contexts
        // 递归寻找子Module是否有要注册的。
        moduleDefinition.subModules.forEach { subModule ->
            registerDefinitions(
                subModule,
                moduleDefinition,
                consolidatedPath
            )
        }
    }
```

`startKoin`到此结束，总结一下