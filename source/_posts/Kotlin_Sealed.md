---
title: Sealed
date: 2019/5/31
categories: Kotlin
---

Java中的枚举一般用于`switch`中使得代码更简洁优美。比如根据类型来播放不同的铃声

```ko
enum class RingerType {
    Outgoing,
    Incoming,
    Busy
}
```

不过枚举类有个缺点，如果想增加额外的数据就会变成这样，这还只是加了一个String类型，额外带个自定义的类型会更加复杂，并且对于`NotFound`来说在构造的时候不需要任何参数。

```kot
enum class RingerType(s: String) {
    Outgoing("R.raw.outgoing"),
    Incoming("R.raw.outgoing"),
    Busy("R.raw.outgoing"),
    NotFound("")
}
```

这种情况则可以使用`sealed class`来描述，如果有额外的数据结构用`data`来修饰，没有任何参数的则可以直接用`class`

```kot
sealed class RingerType {
    class NotFound : RingerType()
    data class Outgoing(val uri: String) : RingerType()
    data class Incoming(val uri: String) : RingerType()
    data class Busy(val uri: String) : RingerType()
}
```

因此kotlin官网上说`sealed class`是枚举的一种延展或者加强版

```
They are, in a sense, an extension of enum classes.
```

