---
title: 使用Jansson产生的引用计数bug
date: 2018/11/26
categories: C++
---

最近在使用Jannson的时候，偶现了一个bug。
```
JanssonTest(83740,0x1003ac380) malloc: *** error for object 0x100600058: incorrect checksum for freed object - object was probably modified after being freed.
```
跟踪了一下代码发现是在free的时候出现的问题。由于是free的时候出现的，因此crash的堆栈还有所不同。
```c
void jsonp_free(void *ptr)
{
    if(!ptr)
        return;

    (*do_free)(ptr);
}
```
可以看到是在执行do_free的时候产生的crash。一开始产生的堆栈一直指向的是将json转成string的一个函数，但是仔细查看了之后发现这一块逻辑没有问题，考虑到是在free的时候出现的问题，就去查看了free相关的函数，最后通过注释排查法找到了原因所在。
```c++
void ProximityRequestBodyBuilder::buildInitConnection(const InitConnectionRequest& request,std::string & requestBody){
    std::string tmp;
    json_t *methodParaJson = json_object();
    
    tmp = request.displayName;
    json_object_set_new(methodParaJson, "displayName", json_string(tmp.c_str()));
    
    json_t *requestJson = wrapper(methodParaJson, "initConnection");

    char *requestStr = json_dumps(requestJson, 0);
    requestBody = std::string(requestStr);
    
    free(requestStr);
    json_decref(requestJson);
    json_decref(methodParaJson);//导致了crash
}

json_t *ProximityRequestBodyBuilder::wrapper(json_t *parameterJson, const std::string &methodName)
{
    json_t *root = json_object();
    
    //wrap token
    json_object_set_new(root, "token", json_string("ed2f"));
    
    //wrap method
    json_t *methodPara = json_object();
    json_object_set_new(methodPara, methodName.c_str(), parameterJson);

    //wrap request
    json_object_set_new(root, "request", methodPara);

    return root;
}
```
在Jannson中使用的引用计数法，也就是哪里用到了这个对象就+1，对应的函数是`json_incref`，如果这个对象之后没用的话要-1，对应的函数是`json_decref`。

但是在调用了`json_decref(requestJson)`之后，`methodParaJson`的引用计数也-1，那么之后再调用`json_decref(methodParaJson)`，就出现了各种问题。

至于两者为什么会有耦合关系，是因为用`requestJson`将`methodParaJson`包装了一次。不过这边既然他自动给我-1，那讲道理在包装的时候应该也自动+1比较合理。

后来check了一下源码才发现不应该用`json_object_set_new`,而应该用`json_object_set`。因为`json_object_set`才是真正+1的地方，而`json_object_set_new`并没有+1，其实感觉new这个关键字有一点误导人。
```c
int json_object_set(json_t *object, const char *key, json_t *value)
{
    return json_object_set_new(object, key, json_incref(value));
}

int json_object_set_new(json_t *json, const char *key, json_t *value)
{
    if(!key || !utf8_check_string(key, strlen(key)))
    {
        json_decref(value);
        return -1;
    }

    return json_object_set_new_nocheck(json, key, value);
}
```

关于这点已经提了issue：https://github.com/akheron/jansson/issues/449
