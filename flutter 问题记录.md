# flutter 问题记录

##安卓降级查询问题

Gradle task bundleRelease failed with exit code 1的意思是你出现了错误，但是没有给你说明错误原因 这是因为你的gradle与gradle-warp版本过高造成的

将buidle.gradle

```
dependencies {
classpath ‘com.android.tools.build:gradle:3.2.1’
}
```
回退到3.2.1

```
gradle-wrapper.properties
distributionUrl=https://services.gradle.org/distributions/gradle-4.10.2-all.zip
```

回退到4.10.2

运行flutter run -v 可以得到具体问题