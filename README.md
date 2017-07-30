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
com.android.build.api.transform.TransformException:
com.android.dex.DexException: Multiple dex files define Lcom/example/Immu
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
 * the branch [java-gradle-plugin-only](https://github.com/tmtron/ImmutablesPackageInfoIssue/tree/java-gradle-plugin-only) does not use those plugins and shows the same error

Software Versions:
* Android Studio 3.0 Canary 5
* Gradle 4.1-milestone-1
* Android gradle plugin: 3.0.0-alpha5

Refrences:
* [StackOverflow question "Immutables-library generates the same immutable class twice"](https://stackoverflow.com/questions/45393034/immutables-library-generates-the-same-immutable-class-twice)
* [StackOverflow question "How to exclude package-info.java files from dex?"](https://stackoverflow.com/a/43026334/6287240)

The Full Build output is in [build.txt](build.txt). See also: [javac-doc](http://docs.oracle.com/javase/7/docs/technotes/tools/windows/javac.html)

**app**: Compiler arguments - simplified:
```
-source 1.7
-target 1.7
-d ..\app\build\intermediates\classes\debug
-encoding UTF-8
-bootclasspath
..\android\SDK\platforms\android-26\android.jar
-g
-sourcepath
-processorpath ..\org.immutables\value\2.5.4\..\value-2.5.4.jar

-XDuseUnsharedTable=true

-classpath
..\org.immutables\value\2.5.4\..\value-2.5.4-annotations.jar;
..\lib\build\libs\lib.jar;
..\appcompat-v7-26.0.0.aar\..\jars\classes.jar;
..\constraint-layout-1.0.2.aar\..\jars\classes.jar;
..\animated-vector-drawable-26.0.0.aar\..\jars\classes.jar;
..\support-vector-drawable-26.0.0.aar\..\jars\classes.jar;
..\support-v4-26.0.0.aar\..\jars\classes.jar;
..\support-media-compat-26.0.0.aar\..\jars\classes.jar;
..\support-fragment-26.0.0.aar\..\jars\classes.jar;
..\support-core-utils-26.0.0.aar\..\jars\classes.jar;
..\support-core-ui-26.0.0.aar\..\jars\classes.jar;
..\support-compat-26.0.0.aar\..\jars\classes.jar;
..\support-annotations\26.0.0\..\support-annotations-26.0.0.jar;
..\constraint-layout-solver-1.0.2.jar

-s ..\app\build\generated\source\apt\debug

..\app\src\main\java\com\example\AndroidIm.java
..\app\src\main\java\com\example\package-info.java
..\app\src\main\java\com\tmtron\immutablespackageinfoissue\MainActivity.java
..\app\build\generated\source\r\debug\android\support\compat\R.java
..\app\build\generated\source\r\debug\android\support\constraint\R.java
..\app\build\generated\source\r\debug\android\support\coreui\R.java
..\app\build\generated\source\r\debug\android\support\coreutils\R.java
..\app\build\generated\source\r\debug\android\support\fragment\R.java
..\app\build\generated\source\r\debug\android\support\graphics\drawable\animated\R.java
..\app\build\generated\source\r\debug\android\support\graphics\drawable\R.java
..\app\build\generated\source\r\debug\android\support\mediacompat\R.java
..\app\build\generated\source\r\debug\android\support\v4\R.java
..\app\build\generated\source\r\debug\android\support\v7\appcompat\R.java
..\app\build\generated\source\r\debug\com\tmtron\immutablespackageinfoissue\R.java
..\app\build\generated\source\buildConfig\debug\com\tmtron\immutablespackageinfoissue\BuildConfig.java
```

The classpath contains the `lib.jar` file which does **NOT** include `JavaIm.java`.
So `..\app\build\generated\source\apt\debug\com\example\ImmutableJavaIm.java` should **NOT** be generated.

**lib**: Compiler arguments - simplified:
```
-source 1.8
-target 1.8
-d ..\lib\build\classes\java\main
-g
-sourcepath
-processorpath ..\org.immutables\value\2.5.4\..\value-2.5.4.jar

-XDuseUnsharedTable=true

-classpath
..\org.immutables\value\2.5.4\..\value-2.5.4-annotations.jar;
..\org.immutables\android-stub\2.5.4\..\android-stub-2.5.4.jar

-s ..\lib\build\generated\source\apt\main

-processorpath ..\org.immutables\value\2.5.4\..\value-2.5.4.jar

..\lib\src\main\java\com\example\JavaIm.java
```

It is strange that `-processorpath` is specified twice, but since it
contains the same value it shouldn't matter.