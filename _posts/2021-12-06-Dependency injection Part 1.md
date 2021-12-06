---
title:  "Dependency injection Part 1" 
toc: true
---

### 目标

- 3W（what、why、how）
- Swinject 概念、scope

---

### What

**控制反转（Inversion of Control）**是一种是面向对象编程中的一种设计原则，用来减低计算机代码之间的耦合度。其基本思想是：借助于“第三方”实现具有依赖关系的对象之间的解耦。

![1957695-bdb55bb000ae89ec](https://upload-images.jianshu.io/upload_images/1957695-bdb55bb000ae89ec.jpg)



**依赖注入 （Dependency injection）**就是将实例变量传入到一个对象中。

**什么是依赖**

如果在 Class A 中，有 Class B 的实例，则称 Class A 对 Class B 有一个依赖。例如下面类 Human 中用到一个 Father 对象，我们就说类 Human 对类 Father 有一个依赖。

```swift
public class Human {
    ...
    let father: Father
    ...
    init() {
        father = Father();
    }
}
```

仔细看这段代码我们会发现存在一些问题：

1. 如果现在要改变 father 生成方式，如需要 Father(name: String) 初始化 father，需要修改 Human 代码；
2. 如果想测试不同 Father 对象对 Human 的影响很困难，因为 father 的初始化被写死在了 Human 的构造函数中；
3. 如果 Father() 过程非常缓慢，单测时我们希望用已经初始化好的 father 对象 Mock 掉这个过程也很困难。



### Why

- 解耦，将依赖之间解耦。

- 因为已经解耦，所以方便做单元测试，尤其是 Mock 测试。



### How

上面将依赖在构造函数中直接初始化是一种 Hard init 方式，弊端在于两个类不够独立，不方便测试。我们还有另外一种 Init 方式，如下：

```swift
public class Human {
    ...
    let father: Father
    ...
    init(father: Father) {
        self.father = father
    }
}
```

上面代码中，我们将 father 对象作为构造函数的一个参数传入。在调用 Human 的构造方法之前外部就已经初始化好了 Father 对象。像这种非自己主动初始化依赖，而通过外部来传入依赖的方式，我们就称为依赖注入。

### 小结

- **控制反转**是一种思想
- **依赖注入**是一种设计模式

---

### Swinject

核心是 `Container` 类。它提供了两类方法，`register` 和 `resolve`。

为了找到在 `resolve` 时，能够找到对应的方法，内部维护了一个叫做`services` 的字典。key 是根据 `serviceType`、`name`、`argumentsType` 确定的。
在 `register` 时，会字典里加入一个条目。在 `resolve` 时，会根据字典，找到对应的 `ServiceEntryProtocol`，然后调用其方法生成一个 component。

#### Container

`register` 方法

```swift
public func _register<Service, Arguments>(
        _ serviceType: Service.Type,
        factory: @escaping (Arguments) -> Any,
        name: String? = nil,
        option: ServiceKeyOption? = nil
    ) -> ServiceEntry<Service> {
        let key = ServiceKey(serviceType: Service.self, argumentsType: Arguments.self, name: name, option: option)
        let entry = ServiceEntry(
            serviceType: serviceType,
            argumentsType: Arguments.self,
            factory: factory,
            objectScope: defaultObjectScope
        )
        entry.container = self
        services[key] = entry

        behaviors.forEach { $0.container(self, didRegisterType: serviceType, toService: entry, withName: name) }

        return entry
    }
```

如上所示，先生成了一个 `ServiceKey`，然后生成了一个 `ServiceEntry`，再把这两个值在字典里对应。

##### ServiceKey

`ServiceKey` 是一个结构体，用来标识注册的一个服务。

##### ServiceEntry

`ServiceEntry` 实现了 `ServiceEntryProtocol` 方法，用来保存对应 `ServiceKey` 生产的实例，以及控制对象的范围(ObjectScopes)。

`resolve` 方法

```swift
public func _resolve<Service, Arguments>(
        name: String?,
        option: ServiceKeyOption? = nil,
        invoker: @escaping ((Arguments) -> Any) -> Any
    ) -> Service? {
        var resolvedInstance: Service?
        let key = ServiceKey(serviceType: Service.self, argumentsType: Arguments.self, name: name, option: option)

        if let entry = getEntry(for: key) {
            resolvedInstance = resolve(entry: entry, invoker: invoker)
        }

        if resolvedInstance == nil {
            resolvedInstance = resolveAsWrapper(name: name, option: option, invoker: invoker)
        }

        if resolvedInstance == nil {
            debugHelper.resolutionFailed(
                serviceType: Service.self,
                key: key,
                availableRegistrations: getRegistrations()
            )
        }

        return resolvedInstance
    }
```

#### Object Scopes

Object Scopes 指定了生成的实例在系统中是如何被共享的。

```text
container.register(Animal.self) { _ in Cat() }
    .inObjectScope(.container)
```

例子如上，每次 `register` 方法，都会返回 `ServiceEntry` 实例，然后调用其 `inObjectScope` 方法，会设置其 `objectScope`。

#####scope 的种类

- Transient
  每次调用 resolve，都会生成新的实例。
- Graph （默认值）
  每次手动调用 resolve，都会生成新的实例。但是在其 factory 方法中生成的实例，是共享的，即如果已经生成过了，就不会重新生成了。用于解决循环依赖
- Container
  被这个 Container 及其子 Container 共享。可以理解为单例。
- Weak
  和 Container 类似，但是Container 并不持有它。如果没有其他引用，这个实例会被销毁，下次重新生成。



### 总结

- 什么是控制反转，注入依赖
- Swinject 的 `Container`、`objectScope` 、`register` 和 `resolve` 概念