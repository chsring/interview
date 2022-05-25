### App编译流程
#### ⼀个Android应⽤从开始编译到打包⼤致上会经历三个步骤：编译前检查--> 编译（资源⽂件和源码）-->打包签名
- 我们以最熟悉的Android Studio开发应⽤为例，⾸先，我们把Gradle构建项⽬从本⽂中剖离开，不讨论具体的Gradle构建是怎么回事，只关注于代码是如何通过⼀系列的Android编译⼯具最终⽣成APK的。
- 编译步骤如下：
- 1. ⾸先在编译阶段，会对资源⽂件⽂件进⾏检查，包括AndroidManifest.xml 、res⽬录下的所有⽂件（⽂件名是否合理）
- 2.在检查通过后，对资源⽂件进⾏处理，⼀次编译⽣成resources.arsc 和 R.java⽂件
- 3.在完成资源编译后，会针对res⽬录下的xml⽂件和AndroidManifest.xml分别进⾏编译，这样处理过的xml⽂件就被简单的“加密”（如果你解压⼀个apk，会发现所有的xml布局⽂件都是乱码），
- 4.最后处理后的res&AndroidManifest.xml&resources.arsc会打包压缩成resources.ap_⽂件，在AndroidStudio的项⽬下/app/build/intermediates/res目录下可以看到编译打包后⽣成的资源打包⽂件。
- 5.如果项⽬中使⽤了NDK来编译native代码，则也会在这个阶段编译并⽣成.so库⽂件。在apkbuilder的时候作为三⽅库⽂件打包进应⽤。
- 6.处理AIDL⽂件，在编译阶段会将.aidl的⽂件编译⽣成相应的java代码供程序调⽤。
- 7.编译⼯程源码，⽣成相应的class⽂件，位于项⽬ /app/build/intermediates/classes ⽬录下。
- 8.在class⽂件⽣成后，如果使⽤了混淆，则会调⽤proguard.jar⽂件对class⽂件和资源⽂件进⾏混淆优化处理，同时⽣成混淆的mapping⽂件，如果没有使⽤混淆，则直接转换所有的class⽂件⽣成classes.dex⽂件。
- 9.打包⽣成APK⽂件。