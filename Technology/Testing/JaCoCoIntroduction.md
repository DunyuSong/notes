# JaCoCo Introduction

-   官网：https://www.eclemma.org/jacoco/
-   源码：https://github.com/jacoco/jacoco
-   JaCoCo is a free code coverage library for Java, which has been created by the EclEmma team based on the lessons learned from using and integration existing libraries for many years.

## 覆盖率定义

-   作为测试如何通过覆盖率保证产品的质量？通常会将测试覆盖率分为两个部分，即“需求覆盖率”和“代码覆盖率”。
-   指令覆盖：计数单元是单个java二进制代码指令，指令覆盖率提供了代码是否被执行的信息，度量完全 独立源码格式。
-   圈复杂度：在（线性）组合中，计算在一个方法里面所有可能路径的最小数目，缺失的复杂度同样表示测 试案例没有完全覆盖到这个模块。



## FAQ

- IDEA 里如何使用Jacoco进行覆盖率查看

1. Run Configurations-->Modify options-->修改 Coverage settings --> Specify alternative  converage runner--> 选择 JaCoCo。

2.  右键运行的时候选择“More Run/Debug”

![idea-runJaCoCo](JaCoCoIntroduction.assets/idea-runJaCoCo.png)






## Reference

- Multi Module Spring Boot集成测试使用JaCoCo生成:https://segmentfault.com/a/1190000023722057

-   https://www.open-open.com/lib/view/open1472174544246.html

-   https://segmentfault.com/a/1190000022259363

-   https://github.com/didi/super-jacoco



