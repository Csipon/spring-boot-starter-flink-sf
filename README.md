# Spring boot starter Flink SF

### Short Description

This spring boot starter can be used only for remote flink modules based on spring boot. 
It will simplify interaction between spring and flink, helps developers to solve routine issues 
and increase function readability

### API

`@Handler` - this annotation could be applied only for methods with following requirements:
1. Public method
2. return value is `CompletableFuture<Void>`
3. Method parameters `Context context, T event` where `T` is any class that can be serialized and deserialized

`DispatchableFunction` - interface that you should use instead of `StatefulFunction`

## Starter in Action

### Step 1 - Create Spring boot Application
I think it shouldn't be an issue for the most developers that familiar with Spring boot.
Guide https://spring.io/guides/gs/spring-boot

### Step 2 - Add starter dependency into maven pom.xml

```xml
<dependency>
    <groupId>com.github.csipon.spring.flink-sf</groupId>
    <artifactId>spring-boot-starter-flink-sf</artifactId>
    <version>0.1.0</version>
</dependency>
```

### Step 3 - Create a function

We created a simple function FooFn that increment COUNT value when receive IncrementEvent.
Also, we can mark it with @Component annotation and now out function is a part of Spring context

```java
@Component
public class FooFn implements DispatchableFunction {

    public static final TypeName TYPE = TypeName.typeNameFromString("namespace/foo");
    public static final ValueSpec<Integer> COUNT = ValueSpec.named("count").withIntType();

    @Handler
    public CompletableFuture<Void> increment(Context context, IncrementEvent event) {
//        do some action with event ...
//        extract value from the context or default
        Integer count = context.storage().get(COUNT).orElse(0);
//        change value
        count++;
//        put changed value into the context
        context.storage().set(COUNT, count);

        return context.done();
    }
}
```

### Step 4 - Add additional handlers

Now we can easily add another handler, let's call it decrement

```java
    @Handler
    public CompletableFuture<Void> decrement(Context context, DecrementEvent event) {
        Integer count = context.storage().get(COUNT).orElse(0);
        count--;
        context.storage().set(COUNT, count);

        return context.done();
    }
```

### Summary 

We've created the Stateful Function that can handle several events(IncrementEvent, DecrementEvent), moreover 
our function is a Spring bean and part of Spring's context, therefore we can use all Spring features for this bean.