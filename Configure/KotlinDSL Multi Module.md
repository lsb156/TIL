# Kotlin DSL Multi Module
코틀린 dsl을 이용한 gradle 멀티 모듈을 설정할때의 설정 파일

## Parent Gradle
``` kotlin
// build.gradle.kts (Parent)
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.72")
    }
}

plugins {
    id("org.springframework.boot") version "2.3.3.RELEASE" apply false
    id("io.spring.dependency-management") version "1.0.10.RELEASE"

    kotlin("jvm") version "1.3.72"
    kotlin("plugin.spring") version "1.3.72"
    kotlin("plugin.jpa") version "1.3.72"
}

allprojects {
    group = "com.ssabae.multi_module_test"
    version = "1.0.0"

    tasks.withType<JavaCompile> {
        sourceCompatibility = "1.8"
        targetCompatibility = "1.8"
    }

    tasks.withType<KotlinCompile> {
        kotlinOptions {
            freeCompilerArgs = listOf("-Xjsr305=strict")
            jvmTarget = "1.8"
        }
    }
}

subprojects {
    repositories {
        mavenCentral()
    }

    apply {
        plugin("io.spring.dependency-management")
    }
}
```


## Children (Main Method가 존재)
```
plugins {
    id("org.springframework.boot")

    kotlin("jvm")
    kotlin("plugin.spring")
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-thymeleaf")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    runtimeOnly("com.h2database:h2")

    testImplementation("org.springframework.boot:spring-boot-starter-test") {
        exclude(group = "org.junit.vintage", module = "junit-vintage-engine")
    }
    testImplementation("org.springframework.security:spring-security-test")
}
```

## Children (Main Method가 없는 모듈)

Main method가 없이 application이 존재하지 않는 모듈을 만들 떄에는
`bootJar.enabled = false` 을 설정하여준다.
보통 여러 모듈에서 사용될 Core 모듈이나 Entity를 정의해두는 모듈로 사용된다.
```
import org.springframework.boot.gradle.tasks.bundling.BootJar

plugins {
    id("org.springframework.boot")
    id("org.flywaydb.flyway") version "6.5.5"
    kotlin("jvm")
    kotlin("plugin.spring")
}

/** main 메소드가 없는 경우 */
val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false
jar.enabled = true

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")

    implementation("org.flywaydb:flyway-core")
    runtimeOnly("com.h2database:h2")
}

flyway {
    url = {DATABASE_URL}
    user = {DATABASE_PASSWORD}
}
```
