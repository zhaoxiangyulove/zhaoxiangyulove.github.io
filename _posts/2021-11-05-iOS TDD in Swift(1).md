---
title:  "iOS TDD in Swift(1)" 
---

---

如果实际开发中，遇到的最多的两种坏现象：

- 为了尽快提测，目前大多时候是代码提测后才开始写 Unit Test Case (后边简称 UT)，也就是 QA 和 UT 几乎同时进行，就会出现在写 UT 时发现代码被乱放，需要调整位置，这可能会引起逻辑改动，导致了需要回测，这就浪费了开发和测试更多的时间。
- 当写完需求再去写 UT，有可能当前是通过的，但如果后边有人在函数里加了不影响 UT 的代码呢，如果没有Test-driven development(后边简称 TDD) 能有效的解决上边两个影响本人效率的问题，能让我写出 testable 和 reasonable 的 code，这就够了。

## Section I: Hello, TDD

### 1. Introduction To TDD

#### 1.1 What is TDD

TDD 是一种通过迭代地进行许多由支持测试的小变更来开发软件的迭代方式。

<img src="https://assets.alexandria.raywenderlich.com/books/itdd/images/1fd7eb6808587a2d64b149794f3228d347d9d62c10d15dba04ce068afce474aa/original.png" alt="img" style="zoom: 25%;" />

这个是 TDD 循环，TDD 能确保彻底并精确地测试你的代码，因为你的代码是被 Test 驱动的。

分为四步:

1. Write a failing test
2. Make the test pass
3. Refactor
4. Repeat

表面上看，TDD 似乎非常简单，实际也是如此。

#### 1.2 Why should you use TDD

#### 1.3 What should you test

#### 1.4 When should you use TDD

#### 1.5 Key points

### 2. Introduction To TDD Cycle

#### 2.1Getting Started

#### 2.2 Red: Write a falling test

#### 2.3 Green: Make the test pass

#### 2.4 Refactor: Clean up your code

#### 2.5 Repeat: Do it again

#### 2.6 Challenge

#### 2.7 Key points









