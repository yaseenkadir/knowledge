Dependency Injection (DI)

Provide a component its dependencies directly instead of having the component create the
dependencies.

```java
public class Foo {
  private Bar bar;
  public Foo() {
    bar = new Bar();
  }

  public String foo() {
    return "foo" + bar.bar();
  }
}

public class Bar {
  public String bar() {
    return "bar";
  }
}
```

In the example above, `Foo` depends on class `Bar`.

The example above does not use Dependency Injection. This presents the following challenges:
* Cannot easily swap out `Bar` for another implementation.
* Foo needs to know how to construct instances of `Bar`
* If `Bar` was an interface `Foo` would depend on a specific implementation (also see Dependency
Inversion) TODO(Yaseen) Add docs for Dependency Inversion

These problems make it much more difficult to swap out implementations e.g. for testing or different
environments. `Foo` also needs to know how to construct instances of `Bar` which means that if
`Bar`'s dependencies change, every place that `Bar` is instantiated will also need to change.

`Foo` can be rewritten so that its `Bar` dependency is injected.

```java
public class Foo {
  private Bar bar;
  public Foo(Bar bar /* bar is being injected via the constructor */) {
    this.bar = bar;
  }

  public String foo() {
    return "foo" + bar.bar();
  }
}
// Bar is the same
```

By injecting the dependency
* It is easier to mock out dependencies for testing
* Consumers of a dependency don't need to updated when the instantiation logic of a dependency
changes
* It is easy swap out implementations of `Bar` without updating `Foo` (see Dependency Inversion)

## Dependency Injection Frameworks
Dependency Injection != Dependency Injection Frameworks (also known as Inversion of Control
Containers)

Dependency Injection is a software development craft technique.

DI frameworks typically allow developers to specify how dependencies should be created and
automatically pass the dependencies to the components that need them.

In software systems that have many dependencies DI frameworks reduce the amount of boilerplate code
needed to instantiate dependencies and pass them to components.

Imagine if `Bar` dependend on `Baz` but `Baz` it depended on `Qux` and `Qux` depended on `Corge`,
without using injecting dependencies `Foo`s constructor would grow quite a lot.

```java
public class Foo {
  private Bar bar;
  public Foo() {
    Corge corge = new Corge();
    Qux qux = new Qux(corge);
    Baz baz = new Baz(qux);
    // Finally Bar is created
    this.bar = new Bar(baz);
  }
}
```

Dependencies are often not as simple as above. For example, in order to instantiate `Bar` it may be
necessary to 
1. Read data from a file
2. Make a network request to an external system
3. Perform decryption of some data

`Bar` may also be used by other components. Without dependency injection, this complicated
instantiation logic will need to be duplicated in many places.

A dependency injection framework can make it easy to wire up all these dependencies automatically.

For example, with the Spring Framework developers can write a configuration such as
```java
public class Config {
  @Bean
  public Foo foo(Bar bar) {
    return new Foo(bar);
  }

  @Bean
  public Bar bar() {
    return new Bar();
  }
}
```

`@Bean` is used to mark objects that should be instantiated and managed by the Spring Inversion of
Control (IoC) container.

The Spring IoC container reads configurations and automatically instantiates the dependencies
and wires them together. It does this by doing something similar to

1. Read configuration file
2. See the `Foo` bean and the method to instantiate it
3. In order to instantiate `Foo` it sees that it depends on `Bar`
4. Spring looks for a `Bar` bean
5. It finds how to declare the `Bar` bean and calls `bar()`
6. It saves the resulting `Bar` bean internally
7. It calls `foo(bar)` and passes the `bar` object from above
8. `Foo` is created

Other dependency injection frameworks operate in similar ways. Typically, developers
1. Write the logic for instantiating a dependency
2. Declare what dependencies should be injected into a component
