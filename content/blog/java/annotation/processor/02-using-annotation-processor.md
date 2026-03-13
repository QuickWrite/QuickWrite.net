+++
date = '2026-03-13T01:58:18+01:00'
draft = false
title = 'Using an Annotation Processor'
+++

When an annotation processor is defined it does not do anything. 
It is just some Java code that is the same as any other Java code.

To use the annotation processor it has to be defined in the `META-INF/services` folder in
`javax.annotation.processing.Processor`. This registers the annotation processor so that Java knows that it even exists.
To actually use the annotation processor, it has to be loaded into the compiler.

## Requirements for using an Annotation Processor
An annotation processor can only be used if the source code has already been compiled. 
This means that the annotation processor has to be compiled **before** the code that 
has to be processed is being compiled.

This is being done using modules. The annotation processor is being implemented in a different Java module that is being
compiled **before** the main codebase. So like this:

```
┌──────────┐                                  
│          │                                  
│   Core   ├───────────────────────┐           
│          │                       │           
└─────┬────┘                     Uses          
      │                            │           
    Uses                           │           
      │                            │           
      │                            │           
      ▼                            ▼           
┌──────────┐            ┌─────────────────────┐
│          │            │                     │
│ Codebase │◄─Processes┄┤ AnnotationProcessor │
│          │            │                     │
└──────────┘            └─────────────────────┘
```

This means that first the module `Core` is being compiled, then the module `AnnotationProcessor` and then at 
the end the module `Codebase`.
This also means that the module `Core` cannot depend on anything and also cannot have the `AnnotationProcessor`.

## Using an Annotation Processor
### With Maven
If an annotation processor exists, it has to be explicitly used at compile time.
In Maven, it is possible to add a module of annotation processors to the compilation 
steps using the `maven-compiler-plugin`:
```xml
<plugins>
    <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.12.1</version>
        <configuration>
            <annotationProcessorPaths>
                <!-- Different modules can be used in here -->
                <path>
                    <groupId><!-- The group id for the module --></groupId> <!-- For example: my.awesome.package -->
                    <groupId><!-- The module --></groupId> <!-- For example: processor -->
                    <version><!-- The version --></version> <!-- For example: 0.1-SNAPSHOT -->
                </path>
            </annotationProcessorPaths>
        </configuration>
    </plugin>
</plugins>
```
If an annotation processor is now added in the `<paths>`, it will now automatically be used every time the project gets
compiled using Maven. If it gets compiled using other means it won't be executed.

### With Gradle
To use the annotation processor with Gradle, it is as simple as using the `annotationProcessor` function directly:
```kt
// Other stuff

dependencies {
    implementation(project(":core"))

    annotationProcessor(project(":annotation-processor"))
}
```
