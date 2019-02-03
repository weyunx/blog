---
layout:     post
title:      Spring Boot 单元测试二三事
subtitle:   All You Need To Know About Unit Testing with Spring Boot
author:     WeYunx
header-style: text
catalog: true
tags:
    - Java
    - Spring boot
    - Test

---

> 本文翻译自：https://reflectoring.io/unit-testing-spring-boot/
>
> 原文作者：Tom Hombergs

写好单元测试是一门技术活，不过好在我们现在有很多框架来帮助我们学习。

本文就为您介绍这些框架，同时详细介绍编写优秀的 Sping Boot 单元测试所必需的技术细节，

我们将了解如何以可测试的方式创建 Spring bean，然后讨论 Mockito 和 AssertJ 的使用，这两个库在默认情况下都集成在 Spring Boot 里。

需要注意的是本文只讨论单元测试，组装测试、web 层测试和持久层测试会在后面的文章里讨论。

# 依赖

在本文中，我们将使用 JUnit Jupiter (JUnit 5), Mockito, and AssertJ，同时还会引入 Lombok 来省去一些繁复的工作。

```groovy
compileOnly('org.projectlombok:lombok')
testCompile('org.springframework.boot:spring-boot-starter-test')
testCompile 'org.junit.jupiter:junit-jupiter-engine:5.2.0'
testCompile('org.mockito:mockito-junit-jupiter:2.23.0')
```

`spring-boot-starter-test` 默认引入了 Mockito and AssertJ，对于 Lombok 则需要我们自己手工引入。

# 不要使用 Spring 进行单元测试

看一下下面的「单元」测试，是用来测试 `RegisterUseCase` 类的一个方法：

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
class RegisterUseCaseTest {

  @Autowired
  private RegisterUseCase registerUseCase;

  @Test
  void savedUserHasRegistrationDate() {
    User user = new User("zaphod", "zaphod@mail.com");
    User savedUser = registerUseCase.registerUser(user);
    assertThat(savedUser.getRegistrationDate()).isNotNull();
  }

}
```

我们去执行这个测试类，花了大概 4.5 秒的时间，原因仅仅是因为计算机要为它去运行一个空的 Spring 项目。

但是，**一个好的单元测试应该是毫秒级的**，否则这会影响「test / code / test」的工作方式，这也就是测试驱动开发的思想 (TDD)。即使我们不做 TDD，在编写测试上花了太多时间也会影响我们的开发思路。

其实，上面的测试方法实际执行只花费了几毫秒，剩下的 4.5 秒全部花费在了 `@SpringBootRun` 上，因为 Spring Boot 需要启动整个 Spring Boot 应用。

**也就是说，我们启动整个应用，耗费了大量资源，仅仅是去为了测试一个方法**，当我们的应用未来越来越大的时候，那将耗费更久的时间去启动。

所以，为什么不要用 Spring Boot 来做单元测试呢？接下来，本文会讨论如何不用 Spring Boot 来进行单元测试。

# 创建测试类

通常，我们可以有如下方法来让我们的 Spring beans 更容易进行测试。

## 不要注入

首先我们先看一个错误的例子：

```java
@Service
public class RegisterUseCase {

  @Autowired
  private UserRepository userRepository;

  public User registerUser(User user) {
    return userRepository.save(user);
  }

}
```

然而这个类还是必须通过 Spring 才能执行，因为我们无法绕过 `UserRepository` 这个实例。就像前面提到的，我们必须换一种方法，不使用 `@Autowired` 来注入 `UserRepository`。

**知识点：不要注入**

## 写一个构造器

我们看一下不使用 `@Autowired` 的写法：

```java
@Service
public class RegisterUseCase {

  private final UserRepository userRepository;

  public RegisterUseCase(UserRepository userRepository) {
    this.userRepository = userRepository;
  }

  public User registerUser(User user) {
    return userRepository.save(user);
  }

}
```

这个版本使用构造器来引入 `UserRepository` 实例。在单元测试中，我们可以像这样来构建一个实例。

Spring 会自动的使用构造器来实例化一个 `RegisterUseCase` 对象。需要注意的是，在 Spring 5 之前，我们需要`@Autowired` 注解来让构造器生效。

同样需要注意的是 `UserRepository` 字段现在是 `final`，这样在整个应用的生命周期里，它都将是个常量，这可以避免编码错误，因为我们如果忘记初始化字段，编译的时候就会报错。

## 减少繁复的代码

使用 Lombok 的 [`@RequiredArgsConstructor`](https://projectlombok.org/features/constructor) 注解，可以让构造器的写法更简洁：

```java
@Service
@RequiredArgsConstructor
public class RegisterUseCase {

  private final UserRepository userRepository;

  public User registerUser(User user) {
    user.setRegistrationDate(LocalDateTime.now());
    return userRepository.save(user);
  }

}
```

现在我们的测试类就很简洁，没有冗余繁复的代码：

```java
class RegisterUseCaseTest {

  private UserRepository userRepository = ...;

  private RegisterUseCase registerUseCase;

  @BeforeEach
  void initUseCase() {
    registerUseCase = new RegisterUseCase(userRepository);
  }

  @Test
  void savedUserHasRegistrationDate() {
    User user = new User("zaphod", "zaphod@mail.com");
    User savedUser = registerUseCase.registerUser(user);
    assertThat(savedUser.getRegistrationDate()).isNotNull();
  }

}
```

不过我们还有一点遗漏，就是如何去模拟 `UserRepository` 实例，因为我们不想去真正的去执行，因为它可能需要去连接数据库。

# 使用 Mockito

现行的标准模拟库是 Mockito，它提供了至少两种方式来模拟 `UserRepository` 。

## 直接调用

第一种方法就是直接使用 Mockito：

```java
private UserRepository userRepository = Mockito.mock(UserRepository.class);
```

这个创建一个对象，看起来和 `UserRepository` 一样。**默认的情况下，这个类什么也不会做，如果调用有返回值的方法，也只会返回 null。**

我们的测试现在会是失败，在 `assertThat(savedUser.getRegistrationDate()).isNotNull()` 这儿报 `NullPointerException` 空指针异常，因为 `userRepository.save(user)` 只会返回 `null`。

所以，我们需要告诉 Mockito，当 `userRepository.save()` 被调用的时候需要有返回值，所以我们使用静态的 `when` 方法：

```java
@Test
void savedUserHasRegistrationDate() {
  User user = new User("zaphod", "zaphod@mail.com");
  when(userRepository.save(any(User.class))).then(returnsFirstArg());
  User savedUser = registerUseCase.registerUser(user);
  assertThat(savedUser.getRegistrationDate()).isNotNull();
}
```

这样 `userRepository.save()` 会返回一个对象，其实这个对象和传入参数的对象一摸一样。

Mockito 具有一整套的测试方案，可以用来模拟、匹配参数以及识别方法的调用，更多资料可以参考[这里](https://static.javadoc.io/org.mockito/mockito-core/2.23.4/org/mockito/Mockito.html)。

## 使用 `@Mock`

此外还可以用 `@Mock` 注解来模拟对象，它需要和 `MockitoExtension` 组合使用。

```java
@ExtendWith(MockitoExtension.class)
class RegisterUseCaseTest {

  @Mock
  private UserRepository userRepository;

  private RegisterUseCase registerUseCase;

  @BeforeEach
  void initUseCase() {
    registerUseCase = new RegisterUseCase(userRepository);
  }

  @Test
  void savedUserHasRegistrationDate() {
    // ...
  }

}
```

`@Mock` 注解会指定字段将被注入到 mock 对象，`@MockitoExtension`  会告诉 Mockito 去扫描 `@Mock` 注解，因为 JUnit 不会自动去执行。

这其实和直接手工执行 `Mockito.mock()` 的结果一样，只是使用习惯的区别。不过使用 `MockitoExtension`   我们的测试就可以绑定到测试框架里。

需要说明的是我们可以在  `registerUseCase` 字段上使用 `@InjectMocks` 注解来替代手工构造一个 `RegisterUseCase` 对象，Mockito 会帮我们自动构造对象，如：

```java
@ExtendWith(MockitoExtension.class)
class RegisterUseCaseTest {

  @Mock
  private UserRepository userRepository;

  @InjectMocks
  private RegisterUseCase registerUseCase;

  @Test
  void savedUserHasRegistrationDate() {
    // ...
  }

}
```

# 让断言更直白

另一个 Spring Boot 自带的测试支持库是 [AssertJ](http://joel-costigliola.github.io/assertj/)，上面的例子里，在实现断言的时候已经用到了：

```java
assertThat(savedUser.getRegistrationDate()).isNotNull();
```

不过我们想让写法变得更直白好理解，比如：

```java
assertThat(savedUser).hasRegistrationDate();
```

通常，我们可以做小改动就可以让代码变得更容易理解，所以我们新建一个自定义的断言对象：

```java
public class UserAssert extends AbstractAssert<UserAssert, User> {

  public UserAssert(User user) {
    super(user, UserAssert.class);
  }

  public static UserAssert assertThat(User actual) {
    return new UserAssert(actual);
  }

  public UserAssert hasRegistrationDate() {
    isNotNull();
    if (actual.getRegistrationDate() == null) {
      failWithMessage("Expected user to have a registration date, but it was null");
    }
    return this;
  }
}
```

这样，我们调用  `UserAssert` 类的 `assertThat` 方法，而不是直接从 Assertj 库里调用。

创建自定义的断言看起来需要很多的工作量，但其实也就是几分钟的事。我相信这几分钟的工作，绝对是值得的，即使是让代码看起来更直白容易理解。**测试代码我们只会写一次，然后其他人（包括我在以后）都只是去读这段代码，然后是反反复复的去修改这段代码，直到产品消亡。**

如果还有疑问，可以参考 [Assertions Generator](http://joel-costigliola.github.io/assertj/assertj-assertions-generator.html)。

# 结论

我们可能有种种的理由在 Spring 里进行测试，但是对于一个普通的单元测试，可以这么做，但是没有必要。随着以后应用越来越庞大，启动时间越来越长，可能还会带来问题。所以，我们在写单元测试的时候，应该以一种更简单的方式去构建 Sprnig bean。

[Spring Boot Test Starter](https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-test&core=gav) 附带了 Mockito 和 AssertJ 作为测试依赖库，所以尽可能的使用这些测试库来做更好的单元测试吧。

所有的代码可以在[这里](https://github.com/thombergs/code-examples/tree/master/spring-boot/spring-boot-testing)找到。

> 如果发现译文存在错误或其他需要改进的地方，欢迎斧正。