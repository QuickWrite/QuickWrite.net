+++
date = '2026-03-13T01:38:18+01:00'
draft = false
title = 'Creating an Annotation Processor'
+++

Annotation processors in Java are a way to hook into the compiler toolchain and can do the following:
1. They can statically check the codebase onto anything they desire as long as there is an annotation present.
2. They can generate code based on the annotations that were provided.
3. They can fail the build process if something is wrong.

They cannot change any file and/or append anything into anything. They can only add new files or check if anything 
else is incorrect.

This means that annotation processors do have the ability to check if constraints are met and also have
the ability to generate code/other resources that the programmer then does not have to write.

## Creating an Annotation Processor
An annotation processor can be created by doing two things:
1. Creating a class that extends `AbstractProcessor` and contains the necessary metadata 
   on which annotations it should process.
2. Registering this processor in the `META-INF/services/javax.annotation.processing.Processor`
   file by adding the qualified name.

A small processor could look like this:

```java
import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.SupportedAnnotationTypes;

@SupportedAnnotationTypes("my.awesome.package.MyAnnotation")
public class TestProcessor extends AbstractProcessor {
    @Override
    public boolean process(final Set<? extends TypeElement> annotations, final RoundEnvironment roundEnv) {
        // Do something with these annotations that were specified.
        
        return true; // Claim the annotations
    }
}
```

It is possible to not specify the `@SupportedAnnotationTypes` annotation, but then the method that relies
on that annotation has to be overwritten.

There is also another method that is always getting called once when the processing starts:
```java
@Override
public synchronized void init(final ProcessingEnvironment processingEnv) {
    super.init(processingEnv);
    
    // Your own code
}
```

## The process method
The process method is **not** getting called just once. It is being called as often as it needs to be called.
It is also not possible to rely on the idea that every annotation already exists in this round.

This is the case because annotation processors are being called (in no particular order) in rounds. This is
always the case if there is new source code available that was generated in the previous round.

----

So for example if we have an annotation processor `A` and another annotation processor `B` and these are
defined like this:
- The processor `A` always generates a source file that is a class annotated with `@AB` if it finds a `@AA`.
- The processor always checks if every `@AB` is defined on a class and not an interface.

This would mean that if there is the annotation `@AA`, the annotation processor `A` would be called in the
first round of processing (mind that the annotation processor `B` wouldn't as there is no annotation `@AB`).<br>
Then the annotation processor `A` would generate a class with the annotation `@AB` and the round would end.

After that the generated source file would be processed by Java, and it would see that there is another annotation.
This means that it initiates another round where `B` is being called. It is checking if the annotation is correctly
placed _(which it is)_ and return. Now the Java compiler sees that it does not have anything anymore to give to the
annotation processors so it goes into another round. This round is the last round where all annotation processors
are being called again with the information that this is a finish round and the annotation processors can still
check stuff as well as generate code; but there won't be any other round.

<!--{{< figure 
    src="/img/blog/java/annotation/processor/java-compiler-annotation-processor.svg"
    alt="The way Java handles annotation processors"
    width="50%"
    class="figure-center"
    caption="How the Java Compiler uses Annotation Processors"
>}}-->

```
         ●
         │                           
         │                           
         ▼                           
╭─────────────────╮                  
│                 │                  
│     process     │   Repeat─┐       
│                 │          │       
╰────────┬────────╯          │       
         │                   │       
         │                   │       
         │                   │       
         │                   │       
         ▼                   │       
╭─────────────────╮          │       
│                 │          │       
│     if_state    │   ├───Else────┐  
│                 │          │    │  
╰────────┬────────╯          │    │  
         │                   │    │  
  New annotations            │    │  
         │                   │    │  
         │                   │    │  
         ▼                   │    ▼  
╭─────────────────╮          │    ●
│                 │          │
│   annotations   │   ├──────┘
│                 │
╰─────────────────╯
```

----

This means that it is not possible to have every annotation in the same round and it is also not possible to assume
that another annotation processor has been executed before. (Lombok is an exception to this rule as it is required to
execute before some other processors as it generates code that they rely on, but the Java compiler does not know about 
that. This is because Lombok changes the Class file, which should not be done.)

To mitigate some of these problems, it is possible to store data in between the rounds by creating an attribute in the
annotation processor as the instance remains the same in between the runs.

[Next up: Usind an Annotation Processor](../02-using-annotation-processor)
