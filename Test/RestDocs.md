## Rest Docs

https://github.com/spring-projects/spring-restdocs/blob/master/samples/rest-notes-spring-hateoas/build.gradle
``` groovy
plugins {
    ...
    id "org.asciidoctor.convert" version "1.5.9.2"
}

ext {
    set 'snippetsDir', file("build/generated-snippets")
}

test {
	outputs.dir snippetsDir
}

asciidoctor {
	inputs.dir snippetsDir
	dependsOn test
}

bootJar {
	dependsOn asciidoctor
	from ("${asciidoctor.outputDir}/html5") {
		into 'static/docs'
	}
}

dependencies {
    ...
    testImplementation('org.springframework.boot:spring-boot-starter-test')
    testImplementation('org.springframework.restdocs:spring-restdocs-mockmvc') // (5)
}

```