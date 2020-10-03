# QueryDsl
복잡한 쿼리를 Java 언어로 제공하여줌으로 컴파일 시점에 문법 오류를 찾을 수 있고 복잡한 쿼리를 쉽게 작성할 수 있다.

## 설정
``` kotlin
// build.gradle.kt
plugins {
    id("com.ewerk.gradle.plugins.querydsl") version "1.0.10"
    kotlin("kapt") version "1.3.72"
    idea
}

buildscript {
    repositories {
        maven("https://plugins.gradle.org/m2/")
        mavenCentral()
    }
    dependencies {
        classpath("gradle.plugin.com.ewerk.gradle.plugins:querydsl-plugin:1.0.10")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.61")
    }
}

dependencies {
    implementation("com.querydsl:querydsl-jpa")
    kapt("com.querydsl:querydsl-apt:4.2.2:jpa")
    kapt("org.hibernate.javax.persistence:hibernate-jpa-2.1-api:1.0.2.Final")
}

idea {
    module {
        val kaptMain = file("build/generated/source/kapt/main")
        sourceDirs.add(kaptMain)
        generatedSourceDirs.add(kaptMain)
    }
}
```
- `com.querydsl:querydsl-apt` : Code Generation Library
- `com.querydsl:querydsl-jpa` : 실제 JPA 쿼리를 간편하게 사용하는 Library


여기서 중요한 점 은 annotation processing을 할 수 있도록 `kapt`를 사용하는 것과, `querydsl-apt` `dependency` 마지막에 `:jpa`를 붙여주는 것이다.

`gradle compileKotlin` 명령어를 이용하여 Q 파일을 생성한다.


