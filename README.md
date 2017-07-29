# ImmutablesPackageInfoIssue

This project reproduces a build issue when we have a multi-module project using the [immutables.io](http://immutables.github.io/) annotation processor and a java `package-info.java` file.  

The project consists of 2 modules:
* app: android application
  * `@Value.Immutable` is used in [AndroidIm.java](/app/src/main/java/com/example/AndroidIm.java)
* lib: java module
  * `@Value.Immutable` is used in [JavaIm.java](lib/src/main/java/com/example/JavaIm.java)

Using [immutables.io](http://immutables.github.io/) in both modules basically works.  
But when we add a `package-info.java` file (to the `com.example` package in the `app` module), the compilation fails with this error:
```
> com.android.build.api.transform.TransformException: com.android.dex.DexException: Multiple dex files define Lcom/example/Immu
tableJavaIm$1;
```

The Dex error occurs, because `ImmutableJavaIm.java` is generated twice:
* once in the lib project (expected):  
  `\build\generated\source\apt\main\com\example\ImmutableJavaIm.java`
* and **ALSO** in the app project (should not happen):  
  `\build\generated\source\apt\debug\com\example\ImmutableJavaIm.java`

Only currently known workaround is to delete the `package-info.java` file (or clear/comment out its contents)

Notes:
 * the project uses the gradle plugins: `idea` and `net.ltgt.apt`
 
Software Versions:
* Android Studio 3.0 Canary 5
* Gradle 4.1-milestone-1
* Android gradle plugin: 3.0.0-alpha5

Refrences:
* [StackOverflow question "Immutables-library generates the same immutable class twice"](https://stackoverflow.com/questions/45393034/immutables-library-generates-the-same-immutable-class-twice)
* [StackOverflow question "How to exclude package-info.java files from dex?"](https://stackoverflow.com/a/43026334/6287240)
