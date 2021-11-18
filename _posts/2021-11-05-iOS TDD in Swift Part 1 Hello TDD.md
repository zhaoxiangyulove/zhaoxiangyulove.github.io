---
title:  "iOS TDD in Swift Part 1 Hello TDD" 
toc: true
---

---

我在开发中，遇到的两种奇怪现象：

- 为了尽快提测，经常提测后才开始写 Unit Test Case (后边简称 UT)， QA 和 UT 几乎同时进行。在写 UT 时发现自己维护的类里出现了乱码七糟的代码（实际更糟糕），需要重新调整位置，这种操作可能会引起逻辑改动，导致了需要回测，浪费了开发和测试更多的时间，为了提前那几天提测，整体花费的时间可能更多。
- 不及时更新 UT 的问题，当写完需求再去写 UT，当时是通过的，但随着需求迭代，有人在函数里加了不影响 UT 的代码，由于不强制在写代码前写好 Test Case，时间久了就会出现 UT 无法使用的情况。

Test-driven development(后边简称 TDD) 正好能解决上边两个现象，那么它是如何避免的呢，通过这篇来讲一下 TDD 和循环，用一个小的实践来演示 TDD 开发。

### 1. TDD

#### 1.1 TDD 是什么

TDD 是一种通过迭代地进行许多由支持测试的小变更来开发软件的迭代方式。这个是 TDD 循环

<img src="https://assets.alexandria.raywenderlich.com/books/itdd/images/1fd7eb6808587a2d64b149794f3228d347d9d62c10d15dba04ce068afce474aa/original.png" alt="img" style="zoom: 25%;" />

TDD 循环分为四步:

1. 写一个会失败的 test。
2. 编写代码让这个 test 通过。
3. 重构。
4. 重复。

你的代码是被 test 驱动出来的，所以 TDD 能彻底并精确地测试你的代码。TDD 似乎非常简单，实际也是如此。

#### 1.2 为什么要使用 TDD，为什么要写 UT

TDD 是确保你的代码执行成功和将来执行依然成功最好的方式。你可能认为不使用 TDD，不写 UT，你也能写出能够上线的代码。当然可以，短期很容易做到，但是长期下来，逻辑逐渐复杂，修改 bug 要阅读的代码消耗的时间将指数增长。当然大环境都是项目排期紧，很难有时间写 UT，更别说 TDD 了，最终还是看你是面向信仰编程还是面向老板编程了。

你可以完成业务开发后再写 UT，或者直接手动测试你的代码。那为什么 TDD 比其他的方式好？

好的 test 能保证你的 App 按照期望运行，然而不是所有的 Test 都是"好的"。好的 test 应该是可失败的、可重复的、快速运行的和可维护的。TDD 提供了方法来确保你的 test 是好的：

- 第一步就是写一个会失败的 test，不失败的 Test 只是在浪费 CPU。
- 当你新的 test 通过时，需要确保所有之前的 tests 都能通过，这就确保你的 tests 是可重复的。你不能只执行你正在开发的 test，建议执行全部 tests。
- 由于要经常执行所有的 tests，所以要确保 test 是快速执行的，如果太久，就没人会执行所有的 tests。
- 当你在重构时，你需要同时更新了生产代码和测试代码，你需要不断地保持它们到最新。
- 随着代码的迭代，你需要确保你的代码是可测试的。如果在开发后再写 UT，那么可能会因此做一些重构来让 test 覆盖。

你会使用一些方法来确保你在写好的 test，你会发现你就在做 TDD！

#### 1.3 应该为哪些代码写 Test

更高的测试覆盖率不代表你的 App 正在被更好的测试，下边是一些建议：

- 建议为自动化不能捕获的代码写 test，比如你自定义类里的函数和大多数你自己写的代码。
- 不建议为生成出的代码写 test，比如生成出 getters 和 setters，你可以相信它能正常 work。
- 不建议为编译器能捕获的问题来写 test。
- 不建议为基础库或第三方库写 test，库的作者应该为这些接口和属性写 test，举个🌰，你不需要为 UIKit 的类来写 tests，然而你需要为你继承的子类写 tests。这些是你自定义出来的代码，所以你应该写对应的 tests。

有两个特殊情况，一种是你想知道一个 framework 是如何 work，那么写一些 tests 是有帮助的，但是这些 tests 不会持续很久。另一种是第三方的代码非常不稳定，所以需要写 tests 来确保，那么你该考虑要不要继续使用这个库了。

##### TDD 花费太多时间！

这是最常见的抱怨了。然而当你习惯使用 TDD 来开发后会比根本不写 tests 更快，因为在后者情况下你最终写了更多的代码。

值得讨论的问题是：开发花费的真实时间！他不止是写一个版本，也包括后边的新需求，变更代码，修复 bugs 等等，在这个长跑过程中，相比于没有 TDD ，TDD 花费了更少的时间，因为它生产更相关的代码，带来更少的 bugs。

另一个需要考虑的问题：线上 bug 的影响。如果 bug 是开发过程中发现，会非常容易修复，但是如果过了几周才发现，你会花费更多时间 debg 代码来寻找根本原因，你的 tests 会帮助你对抗这种情况。

#### 1.4 什么时间开始使用 TDD

如果你的 App 要发布并持续迭代，同时/或者你的工程有复杂逻辑，你最好及时使用 TDD。

如果你只是用来写一个 demo 工程或者临时的工程， 那么你随意。

总之，TDD 是一个工具，你自己来决定什么时候来使用它。

#### 1.5 关键点

你应该了解了 TDD 是什么，为什么使用它，哪些代码应该被测试和什么时间去使用它。

- TDD 提够了写好的 tests 的方式。
- 好的 test 是可失败的、可重复的、快速运行的和可维护的。
- 开发真正的时间花费包含初始开发，添加新需求，修改现有逻辑和修复 bugs 等等。TDD 减少维护成本和保证了 bugs 的可控
- TDD 对于一个长期的项目是非常有帮助的。

### 2. TDD 循环

#### 2.1 准备

在这个小节，你讲继续关注于 TDD ，你可以使用 playground，创建一个 **CashRegister.playground** 。"File" -> "New" -> "Playground"， 选择 "Blank"。

![image-20211110182716628](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/image-20211110182716628.png?raw=true)

删掉所有的代码，开始使用 TDD。

#### 2.2 红色

在你写生产代码前，你需要写一个会失败的 test。

首先你需要建一个 testCase 的类，复制下边的代码到 playground。

```swift
import XCTest

class CashRegisterTests: XCTestCase {

}
```

编译，如果报错 **error: unable to load standard library for target 'arm64-apple-macos11.0'** 需要将 platform 设置为 macOS，运行通过。

接下来在 playground 最下方添加

```swift
CashRegisterTests.defaultTestSuite.run()
```

这是让 playground 去跑 <hl>CashRegisterTests</hl> 下所有的 tests，然而还没有任何 tests，所以在里边添加一个会引起编译报错的 test。

```swift
// 1
func testInit_createsCashRegister() {
  // 2
  XCTAssertNotNil(CashRegister())
}

```

1. 解释下函数命名：

- XCTest：需要所有的测试函数以 <hl>test</hl> 开头。
- test：接下来是函数要测试的名字，这里是 <hl>init</hl>，用 "_"和下个部分区分。
- 最后的部分是期望的输出或结果，这里是 <hl>createsCashRegister</hl> 。

这种命名容易理解同时提供有用的信息，如果 test 失败了，Xcode 会告诉你 test 的类和函数名，通过这种命名方式可以快速了解问题。

2. 你试图初始化一个<hl>CashRegister</hl> 的实例。通过 <hl>XCTAssertNotNil</hl> 表达式来判断，当参数不为 nil 则通过。

![WX20211110-185352](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/WX20211110-185352.png?raw=true)

然而，第七行并不能编译，因为你还没有创建<hl>CashRegister</hl> 的类，在 TDD 里有这样一条规则，编译失败视为 test 失败，所以你已经完成了 TDD 循环的第一步。

#### 2.3 绿色

TDD 中你只能写最少的代码让这个 test 通过，如果你写了比下边的代码更多，你的 test 将会落后于你的生产代码！那最少的代码能解决上边的编译问题的代码是什么？声明一个<hl>CashRegister</hl>类。

在<hl>CashRegisterTest</hl>上方添加下面的代码：

```swift
class CashRegister {
  
}
```

点击执行，你会看到控制台输出：

```swift
Test Suite 'CashRegisterTests' started at 2021-11-10 22:20:45.074
Test Case '-[__lldb_expr_18.CashRegisterTests testInit_createsCashRegister]' started.
Test Case '-[__lldb_expr_18.CashRegisterTests testInit_createsCashRegister]' passed (0.003 seconds).
Test Suite 'CashRegisterTests' passed at 2021-11-10 22:20:45.078.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.003 (0.004) seconds
```

好耶！你已经让 test 通过了，下一步，重构你的代码。

#### 2.4 重构

你将会在重构这一步同时清理你的生产代码和测试代码，这样你能维持它们之间的联系，同时提升你的代码，这里有几件事可以去重构。

- 重复的逻辑。
- 一些无用的注释。这里解释下，你的注释应该去解释为什么会写下边的代码，而不是下边的代码做了什么。后者更应该去拆分为几个合理的函数。。
- 代码坏味道。《重构 改善既有代码的设计》第三章，有详细的介绍，有兴趣可以去读一读。

目前，<hl>CashRegister</hl> 和 <hl>CashRegisterTests</hl> 还没有很多代码，所以无需重构，所以这一步已经完成了。接下来是最重要的一步：重复。

#### 2.5 重复

这一步是 TDD 循环中受益最大的一步。你将在每个 TDD 循环完成一部分功能，通过测试用例来驱动实现逻辑。当你完成所有的需求时，你的 UT 也是完整的。

你已经完成了第一个 TDD 循环，你有一个可以初始化的 <hl>CashRegister</hl> 类，这个类可以添加一些有用的函数。下边是一些 todo：

- 添加初始化函数接受 <hl>availableFunds</hl>。
- 添加 <hl>addItem</hl> 函数，用来添加交易。
- 添加<hl>acceptPayment</hl> 函数。

##### TDDing - init(availableFunds:)

按照 TDD 的步骤，首先要写一个失败的 test，在之前的 test 下面添加代码：

```swift
func testInitAvailableFunds_setsAvailableFunds() {
  // given
  let availableFunds = Decimal(100)
  
  // when
  let sut = CashRegister(availableFunds: availableFunds)
  
  // then
  XCTAssertEqual(sut.availableFunds, availableFunds)
}
```

这个 test 和第一个相比更加复杂了。所以把它拆分成三个部分：**given**、**when** 和 **then** 。用这种方式考虑 UT 很有用：

- 提供一个确认的条件。
- 当一个确认的行为发生时。
- 那么一个预期的结果会出现。

以这个 case 为 🌰，你提供了一个 100 的 <hl>availableFunds</hl>，当你用它创建了一个 sut 实例，那么你期望 <hl>sut.availableFunds</hl> 应和 <hl>availableFunds</hl> 相等。

sut( **system under test**)，在 TDD 中无论你在测试什么都可以使用这个名字。

由于还没有声明 <hl>init(availableFunds:)</hl> ,目前代码仍无法编译。之前说过，编译报错认为是测试失败，所以已经完成了第一步。

接下来添加代码到 <hl>CashRegister</hl>，使编译通过：

```
let availableFunds: Decimal

init(availableFunds: Decimal = 0) {
  self.availableFunds = availableFunds
}
```

现在点击 **Play** 执行所有的 tests ，你将在控制台看到：

```swift
Test Suite 'CashRegisterTests' started at 2021-11-18 17:27:34.168
Test Case '-[__lldb_expr_1.CashRegisterTests testInit_createsCashRegister]' started.
Test Case '-[__lldb_expr_1.CashRegisterTests testInit_createsCashRegister]' passed (0.003 seconds).
Test Case '-[__lldb_expr_1.CashRegisterTests testInitAvailableFunds_setAvailableFunds]' started.
Test Case '-[__lldb_expr_1.CashRegisterTests testInitAvailableFunds_setAvailableFunds]' passed (0.003 seconds).
Test Suite 'CashRegisterTests' passed at 2021-11-18 17:27:34.176.
	 Executed 2 tests, with 0 failures (0 unexpected) in 0.006 (0.008) seconds
```

这里现实两个 tests 都通过了，所以你完成了第二步。

接下来开始清理生产代码和测试代码，首先看下测试代码。

<hl>testInit_createsCashRegister</hl>现在是无效的了，因为已经没有 <hl>init()</hl> 了，这个 test 实际上是调用 <hl>init(availableFunds:)</hl> ，所以直接删除这个 test 。

在看生产代码，添加这个默认值 <hl>0</hl> 是否有必要？为了同时通过 <hl>testInit</hl> 和 <hl>testInitAvailableFunds</hl> 是必要的。但是对于这个类来说呢？

这里需要讨论下设计：

- 如果你想继续保留这个默认值，那你需要添加一个 <hl>testInit_setsDefaultAvailableFunds</hl> ，来检查默认值的情况。

- 或者你可以选择删除默认值。

在这里，假设不需要这个默认值，所以删除掉这个默认值。你的初始化函数变成：

```swift
init(availableFunds: Decimal) {
```

点击 **Play** 执行你剩下的 test ，你会看到它通过了。

在重构完 <hl>init(availableFunds:</hl> 后 <hl>testInitAvailableFunds</hl> 仍然是通过的，表示你的改动没用破坏现有的代码设计，信心倍增，这是 TDD 在重构中带来的一大好处。

现在你完成了重构，接下来你要再次重复这些过程。

##### TDDing addItem

#### 2.6 Challenge

#### 2.7 Key points









