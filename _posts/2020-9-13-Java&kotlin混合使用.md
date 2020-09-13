---
layout:     post                    # 使用的布局（不需要改）
title:      JAVA&Kotlin混合使用配置               # 标题 
subtitle:   导入注解库怎样混合配置 #副标题
date:       2020-9-13              # 时间
author:     JT                      # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Android 
    - Java 
    - Kotlin
    - Idea
---

### 以dagger库导入为例

- 注解类的库都需要这样弄 kapt 导入
#### JAVA
- 目标model的dependencies

```java
implementation 'com.google.dagger:dagger:2.21'
implementation 'com.google.dagger:dagger-android:2.21'
annotationProcessor 'com.google.dagger:dagger-compiler:2.21' 
```

#### Kotlin
- 目标model的dependencies

```groovy
implementation 'com.google.dagger:dagger:2.21'
kapt 'com.google.dagger:dagger-compiler:2.21'
```
#### JAVA和Kotlin混合使用
- 需要在目标build.gradle(module)的最后添加下面代码(module中)

```groovy
apply plugin: 'kotlin-kapt

kapt {
    generateStubs = true
}
```

