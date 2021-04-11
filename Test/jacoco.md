# Jacoco 
테스트 커버리지 측정 도구
```groovy

plugins {
    id 'jacoco'
}

test {
    jacoco {
        destinationFile = file("$buildDir/jacoco/jacoco.exec")
    }
    finalizedBy 'jacocoTestReport'
}

jacoco {
    toolVersion = '0.8.5'
}

jacocoTestReport {
    reports {
        html.enabled true
        xml.enabled true    // sonarqube와 연동시 필요
        csv.enabled false
    }
    finalizedBy 'jacocoTestCoverageVerification'
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            enabled = true              // rule에 대한 조건 사용 여부
            element = "CLASS"

            limit {
                counter = "BRANCH"      // 브랜치 커버리지
                value = "COVEREDRATIO"
                minimum = 0.70
            }

            limit {
                counter = 'LINE'        // 라인 커버리지
                value = 'COVEREDRATIO'
                minimum = 0.70
            }

            limit {
                counter = 'LINE'        // 빈 줄을 제외한 코드의 라인수를 최대 200라인으로 제한
                value = 'TOTALCOUNT'
                maximum = 200
            }

            excludes = [                // 분석에서 제외할 파일 패턴
//                    '*.test.*',
            ]
        }
    }
}

task testCoverage(type: Test) {
    group 'verification'
    description 'Runs the unit tests with coverage'

    dependsOn(':test',
            ':jacocoTestReport',
            ':jacocoTestCoverageVerification')

    tasks['jacocoTestReport'].mustRunAfter(tasks['test'])
    tasks['jacocoTestCoverageVerification'].mustRunAfter(tasks['jacocoTestReport'])
}
```

`./gradlew testCoverage`


> https://blog.leocat.kr/notes/2020/02/02/jacoco-config-jacoco-for-kotlin-and-java-project
> 