# ImmutablesPackageInfoIssue

This project reproduces a build issue when we have a multi-module project using the [immutables.io](http://immutables.github.io/) annotation processor and a java `package-info.java` file.  

The project in this branch `android-project-only` consists of only the
android-app, and it uses the generated `lib.jar` file as
direct dependency.

* app: android application
  * `@Value.Immutable` is used in [AndroidIm.java](/app/src/main/java/com/example/AndroidIm.java)

The `lib.jar` file contains only the generated `.class` files of `JavaIm.java`.
So there is no way for the immutables annotation processor to access
`JavaIm.java`. But still the build generates
`\build\generated\source\apt\debug\com\example\ImmutableJavaIm.java`!?