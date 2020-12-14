# 构建Java和JVM项目

内容

- [介绍](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#introduction)
- [通过源集声明源文件](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_source_sets)
- [管理你的依赖](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_dependency_management_overview)
- [编译代码](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:compile)
- [管理资源](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_resources)
- [运行测试](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:running_java_tests)
- [包装出版](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_packaging)
- [生成API文档](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:generating_javadocs)
- [清理构建](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:cleaning_java_build)
- [构建Java库](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_libraries)
- [构建Java应用程序](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_applications)
- [构建Java Web应用程序](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_webapps)
- [构建Java EE应用程序](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_enterprise_apps)

Gradle使用基于配置的约定方法来构建基于JVM的项目，该方法借鉴了Apache Maven的几种约定。特别是，它对源文件和资源使用相同的默认目录结构，并且可与Maven兼容的存储库一起使用。

我们将在本章中详细研究Java项目，但是大多数主题也适用于其他受支持的JVM语言，例如[Kotlin](https://guides.gradle.org/building-kotlin-jvm-libraries/)，[Groovy](https://docs.gradle.org/4.10.2/userguide/groovy_plugin.html#groovy_plugin)和[Scala](https://docs.gradle.org/4.10.2/userguide/scala_plugin.html#scala_plugin)。如果您没有使用Gradle构建基于JVM的项目的丰富经验，请首先看一下[Java Quickstart](https://docs.gradle.org/4.10.2/userguide/tutorial_java_projects.html#tutorial_java_projects)，因为它将为您提供基本的概述。

## [介绍](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#introduction)

Java项目的最简单构建脚本将应用[Java插件，](https://docs.gradle.org/4.10.2/userguide/java_plugin.html#java_plugin)并可以选择设置项目版本和Java兼容版本：

### [示例：应用Java插件](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_applying_the_java_plugin)

build.gradle

```groovy
plugins {
    id 'java'
}

sourceCompatibility = '1.8'
targetCompatibility = '1.8'
version = '1.2.1'
```

通过应用Java插件，您可以获得许多功能：

- 一`compileJava`，编译下的所有Java源文件任务*的src / main / JAVA*
- 一`compileTestJava`对在源文件任务*的src / test / java下*
- 一个`test`运行从测试任务*的src / test / java下*
- 将来自*src / main / resources*`jar`的已`main`编译类和资源打包到名为*- .jar*的单个JAR中的任务
- `javadoc`为`main`类生成Javadoc的任务

这还不足以构建任何重要的Java项目-至少，您可能会有一些文件依赖性。但它意味着你的构建脚本只需要特定于信息*的*项目。

| ✨    | 尽管示例中的属性是可选的，但我们建议您在项目中指定它们。兼容性选项可缓解使用不同Java编译器版本构建的项目的问题，并且版本字符串对于跟踪项目的进度很重要。默认情况下，项目版本也用于归档名称中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Java插件还将上述任务集成到标准的[基本插件生命周期任务中](https://docs.gradle.org/4.10.2/userguide/base_plugin.html#sec:base_tasks)：

- `jar`附加到`assemble` [ [1](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#_footnotedef_1) ]
- `test` 连接到 `check`

本章的其余部分介绍了根据需要自定义构建的不同方法。稍后，您还将看到如何调整库，应用程序，Web应用程序和企业应用程序的构建。

## [通过源集声明源文件](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_source_sets)

Gradle对Java的支持是第一个引入用于构建基于源代码的项目的新概念的方法：*源代码集*。主要思想是源文件和资源通常按类型进行逻辑分组，例如应用程序代码，单元测试和集成测试。每个逻辑组通常都有其自己的文件依赖项集，类路径等。重要的是，构成源集的文件*不必位于同一目录中*！

源集是一个强大的概念，将编译的几个方面联系在一起：

- 源文件及其位置
- 编译类路径，包括任何必需的依赖项（通过Gradle[配置](https://docs.gradle.org/4.10.2/userguide/dependency_management_terminology.html#sub:terminology_configuration)）
- 放置已编译的类文件的位置

您可以在此图中看到它们之间的相互关系：

![Java SourceSets编译](https://docs.gradle.org/4.10.2/userguide/img/java-sourcesets-compilation.png)

图1.源集和Java编译

阴影框表示源集本身的属性。最重要的是，Java插件会为您或插件定义的每个源集（命名为）以及多个[依赖项配置](https://docs.gradle.org/4.10.2/userguide/java_plugin.html#java_source_set_configurations)自动创建一个编译任务。`compile*SourceSet*Java`

| ✨    | 该`main`源集大多数语言插件（包括Java）会自动创建一个名为的源集`main`，该源集用于项目的生产代码。此源组的特殊之处在于它的名字没有在配置和任务的名称包括在内，因此为什么你只是一个`compileJava`任务，`compileOnly`和`implementation`配置，而不是`compileMainJava`，`mainCompileOnly`和`mainImplementation`分别。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Java项目通常包含源文件以外的资源（例如属性文件），这些资源可能需要处理（例如，通过替换文件中的标记）并打包在最终JAR中。Java插件通过自动创建用于称为每个定义的源集的专用任务处理这种（或在源集）。下图显示了源集如何适合此任务：`process*SourceSet*Resources``processResources``main`

![java资源集处理资源](https://docs.gradle.org/4.10.2/userguide/img/java-sourcesets-process-resources.png)

图2.处理源集的非源文件

与以前一样，阴影框表示源集的属性，在这种情况下，该属性包括资源文件的位置以及将它们复制到的位置。

除了`main`源集之外，Java插件还定义了`test`代表项目测试的源集。该源集由`test`运行测试的任务使用。您可以在[Java测试](https://docs.gradle.org/4.10.2/userguide/java_testing.html#java_testing)一章中了解有关此任务和相关主题的更多信息。

项目通常使用此源集进行单元测试，但如果需要，也可以将其用于集成，验收和其他类型的测试。另一种方法是为其他每种测试类型[定义一个新的源集](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:custom_java_source_sets)，通常是出于以下两个或多个原因：

- 您想使测试彼此分开，以保持美观和可管理性
- 不同的测试类型需要不同的编译或运行时类路径或设置上的其他差异

您可以在Java测试一章中看到这种方法的示例，该示例向您[展示了如何](https://docs.gradle.org/4.10.2/userguide/java_testing.html#sec:configuring_java_integration_tests)在项目中[设置集成测试](https://docs.gradle.org/4.10.2/userguide/java_testing.html#sec:configuring_java_integration_tests)。

您将了解有关源集及其提供的功能的更多信息：

- [自定义文件和目录位置](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:custom_java_source_set_paths)
- [配置Java集成测试](https://docs.gradle.org/4.10.2/userguide/java_testing.html#sec:configuring_java_integration_tests)

## [管理你的依赖](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_dependency_management_overview)

绝大多数Java项目都依赖于库，因此管理项目的依赖关系是构建Java项目的重要组成部分。依赖管理是一个大话题，因此我们将在这里重点介绍Java项目的基础知识。如果您想深入研究细节，请查看[依赖管理](https://docs.gradle.org/4.10.2/userguide/introduction_dependency_management.html#introduction_dependency_management)的[介绍](https://docs.gradle.org/4.10.2/userguide/introduction_dependency_management.html#introduction_dependency_management)。

为Java项目指定依赖项仅需要三点信息：

- 您需要哪个依赖项，例如名称和版本
- 它需要什么，例如编译或运行
- 在哪里寻找

前两个在`dependencies {}`块中指定，第三个在`repositories {}`块中指定。例如，要告诉Gradle您的项目需要3.6.7版的[Hibernate](http://hibernate.org/) Core来编译和运行生产代码，并且要从Maven Central存储库下载该库，可以使用以下片段：

### [示例：声明依赖关系](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_declaring_dependencies)

build.gradle

```groovy
repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.hibernate:hibernate-core:3.6.7.Final'
}
```

三个元素的Gradle术语如下：

- *存储库*（例如：`mavenCentral()`）—在哪里查找声明为依赖项的模块
- *配置*（例如：`implementation`）-命名的依赖项集合，针对特定目标（如编译或运行模块）分组在一起-一种更灵活的Maven范围形式
- *模块坐标*（ex：`org.hibernate:hibernate-core-3.6.7.Final`）—依赖项的ID，通常采用' **：**：** '的形式（或在Maven术语中为' **：**：** '）

您可以[在此处](https://docs.gradle.org/4.10.2/userguide/dependency_management_terminology.html#dependency_management_terminology)找到更全面的依赖项管理术语表。

就配置而言，主要感兴趣的是：

- `compileOnly` —用于编译生产代码所必需的依赖关系，但不应作为运行时类路径的一部分
- `implementation`（取代`compile`）-用于编译和运行时
- `runtimeOnly`（取代`runtime`）-仅在运行时使用，不用于编译
- `testCompileOnly`—与`compileOnly`测试相同
- `testImplementation` —测试相当于 `implementation`
- `testRuntimeOnly` —测试相当于 `runtimeOnly`

您可以在[插件参考一章中](https://docs.gradle.org/4.10.2/userguide/java_plugin.html#sec:java_plugin_and_dependency_management)了解有关它们的更多信息以及它们之间的关系。

请注意，[Java库插件](https://docs.gradle.org/4.10.2/userguide/java_library_plugin.html#java_library_plugin)`api`为编译模块和依赖该模块的任何模块都需要的依赖项创建了一个附加配置。

| ✨    | 为什么没有`compile`配置？Java插件在历史上一直使用该`compile`配置作为编译和运行项目生产代码所需的依赖项。现在已弃用它（尽管它不会很快消失），因为它无法区分影响Java库项目的公共API的依赖项和不影响Java库项目的公共API的依赖项。您可以在[构建Java库中](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_libraries)了解有关此区别的重要性的更多信息。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

我们仅在此处进行了介绍，因此，一旦您熟悉使用Gradle构建Java项目的基础知识，我们建议您阅读专用的依赖管理章节。需要进一步阅读的一些常见方案包括：

- 定义与[Maven](https://docs.gradle.org/4.10.2/userguide/repository_types.html#sub:maven_repo)或[Ivy兼容](https://docs.gradle.org/4.10.2/userguide/repository_types.html#sec:ivy_repositories)的自定义存储库
- 使用[本地文件系统目录中的](https://docs.gradle.org/4.10.2/userguide/repository_types.html#sec:flat_dir_resolver)依赖项
- 使用[变化的版本](https://docs.gradle.org/4.10.2/userguide/declaring_dependencies.html#sub:declaring_dependency_with_changing_version)（例如SNAPSHOT）和[动态的](https://docs.gradle.org/4.10.2/userguide/declaring_dependencies.html#sub:declaring_dependency_with_dynamic_version)（范围）声明依赖项
- 将同级[项目](https://docs.gradle.org/4.10.2/userguide/declaring_dependencies.html#sec:declaring_project_dependency)声明[为依赖项](https://docs.gradle.org/4.10.2/userguide/declaring_dependencies.html#sec:declaring_project_dependency)
- [控制传递依赖及其版本](https://docs.gradle.org/4.10.2/userguide/managing_transitive_dependencies.html#managing_transitive_dependencies)
- 通过[组合构建](https://docs.gradle.org/4.10.2/userguide/composite_builds.html#composite_builds)测试对第三方依赖关系的修复（这是发布到[Maven Local](https://docs.gradle.org/4.10.2/userguide/repository_types.html#sub:maven_local)和从[Maven Local](https://docs.gradle.org/4.10.2/userguide/repository_types.html#sub:maven_local)消费的更好的替代方法）

您会发现Gradle具有丰富的API用于处理依赖关系-一种需要花费时间才能掌握的API，但对于常见情况而言，它很容易使用。

## [编译代码](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:compile)

如果遵循以下约定，则可以同时轻松地编译生产代码和测试代码：

1. 将生产源代码放在*src / main / java*目录下
2. 将您的测试源代码放在*src / test / java下*
3. 在`compileOnly`或`implementation`配置中声明您的生产编译依赖项（请参见上一节）
4. 在`testCompileOnly`或`testImplementation`配置中声明您的测试编译依赖项
5. 运行`compileJava`生产代码和`compileTestJava`测试的任务

其他JVM语言插件（例如[Groovy的](https://docs.gradle.org/4.10.2/userguide/groovy_plugin.html#groovy_plugin)插件）遵循相同的约定模式。我们建议您尽可能遵循这些约定，但是不必这样做。有几个自定义选项，您将在下面看到。

### [自定义文件和目录位置](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:custom_java_source_set_paths)

假设您有一个旧项目，该项目使用*src*目录存储生产代码并*测试*测试代码。传统的目录结构不起作用，因此您需要告诉Gradle在哪里可以找到源文件。您可以通过源集配置来完成。

每个源集都定义其源代码所在的位置，以及类文件的资源和输出目录。您可以使用以下语法覆盖约定值：

#### [示例：声明自定义源目录](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_declaring_custom_source_directories)

build.gradle

```groovy
sourceSets {
    main {
         java {
            srcDirs = ['src']
         }
    }

    test {
        java {
            srcDirs = ['test']
        }
    }
}
```

现在Gradle将只在*src中*直接搜索并*测试*相应的源代码。如果您不想覆盖约定，而只想*添加*一个额外的源目录，该目录可能包含一些您想分开的第三方源代码，该怎么办？语法类似：

#### [示例：以附加方式声明自定义源目录](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_declaring_custom_source_directories_additively)

build.gradle

```groovy
sourceSets {
    main {
        java {
            srcDir 'thirdParty/src/main/java'
        }
    }
}
```

至关重要的是，我们在此处使用此*方法* `srcDir()`来追加目录路径，而设置`srcDirs`属性将替换所有现有值。这是Gradle中的常见约定：设置属性将替换值，而相应的方法将附加值。

您可以在[SourceSet](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.SourceSet.html)和[SourceDirectorySet](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.file.SourceDirectorySet.html)的DSL参考中[看到](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.SourceSet.html)源集上可用的所有属性和方法。请注意，`srcDirs`和`srcDir()`都在`SourceDirectorySet`。

### [更改编译器选项](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#changing_compiler_options)

可通过相应的任务（例如`compileJava`和）访问大多数编译器选项`compileTestJava`。这些任务的类型为[JavaCompile](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.compile.JavaCompile.html)，因此，请阅读任务参考以获取最新，最全面的选项列表。

例如，如果要为编译器使用单独的JVM进程并防止编译失败使构建失败，则可以使用以下配置：

#### [示例：设置Java编译器选项](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_setting_java_compiler_options)

build.gradle

```groovy
compileJava {
    options.incremental = true
    options.fork = true
    options.failOnError = false
}
```

这也是您可以更改编译器的详细程度，禁用字节码中的调试输出以及配置编译器可以在何处找到注释处理器的方式。

在项目级别定义了Java编译器的两个常用选项：

- `sourceCompatibility`

  定义应将源文件视为Java的语言版本。

- `targetCompatibility`

  定义您的代码应在其上运行的最低JVM版本，即，它确定编译器生成的字节码的版本。

如果出于任何原因需要或想要多个编译任务，则可以[创建一个新的源集](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:custom_java_source_sets)，也可以简单地定义一个[JavaCompile](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.compile.JavaCompile.html)类型的新任务。接下来，我们来看设置新的源集。

### [编译和测试Java 6/7](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_cross_compilation)

Gradle只能在Java版本7或更高版本上运行。但是，已经不支持在Java 7上运行Gradle，并且计划在Gradle 5.0中删除对它的支持。弃用对Java 7的支持有两个原因：

- Java 7达到[了使用寿命](http://www.oracle.com/technetwork/java/javase/eol-135779.html)。因此，自2015年4月起，Oracle停止了对Java 7的安全修补程序和升级程序的公开发布。
- 一旦不再支持Java 7（可能是Gradle 5.0），Gradle的实现就可以开始使用针对性能和可用性进行了优化的Java 8 API。

Gradle仍然支持Java 6和Java 7的编译，测试，生成Javadoc并执行应用程序。不支持Java 5。

要使用Java 6或Java 7，需要配置以下任务：

- `JavaCompile` 分叉并使用正确的Java主页的任务
- `Javadoc`使用正确的`javadoc`可执行文件的任务
- `Test`以及`JavaExec`使用正确的`java`可执行文件的任务。

以下示例显示了如何`build.gradle`调整需求。为了能够独立于构建机器，应在每个开发人员机器上用户主目录的`GRADLE_USER_HOME/gradle.properties` [ [2](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#_footnotedef_2) ]中配置旧Java主目录和目标版本的位置，如示例所示。

#### [示例：配置Java 6构建](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_configure_java_6_build)

gradle.properties

```properties
# in $HOME/.gradle/gradle.properties
javaHome=/Library/Java/JavaVirtualMachines/1.7.0.jdk/Contents/Home
targetJavaVersion=1.7
```

build.gradle

```groovy
assert hasProperty('javaHome'): "Set the property 'javaHome' in your your gradle.properties pointing to a Java 6 or 7 installation"
assert hasProperty('targetJavaVersion'): "Set the property 'targetJavaVersion' in your your gradle.properties to '1.6' or '1.7'"

sourceCompatibility = targetJavaVersion

def javaExecutablesPath = new File(javaHome, 'bin')
def javaExecutables = [:].withDefault { execName ->
    def executable = new File(javaExecutablesPath, execName)
    assert executable.exists(): "There is no ${execName} executable in ${javaExecutablesPath}"
    executable
}
tasks.withType(AbstractCompile) {
    options.with {
        fork = true
        forkOptions.javaHome = file(javaHome)
    }
}
tasks.withType(Javadoc) {
    executable = javaExecutables.javadoc
}
tasks.withType(Test) {
    executable = javaExecutables.java
}
tasks.withType(JavaExec) {
    executable = javaExecutables.java
}
```

### [分别编译独立源](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:custom_java_source_sets)

大多数项目至少有两个独立的源集：生产代码和测试代码。Gradle已经将此场景作为其Java约定的一部分，但是如果您有其他来源的话该怎么办？最常见的情况之一是当您进行某种形式或其他形式的单独集成测试时。在这种情况下，自定义源集可能正是您所需要的。

您可以在[Java测试一章中](https://docs.gradle.org/4.10.2/userguide/java_testing.html#sec:configuring_java_integration_tests)看到设置集成测试的完整示例。您可以设置以相同方式担当不同角色的其他源集。问题就变成了：您何时应该定义自定义源集？

要回答该问题，请考虑以下来源：

1. 需要使用唯一的类路径进行编译
2. 生成与`main`和处理不同的`test`类
3. 构成项目的自然组成部分

如果您对3和其他任何一个的回答都是肯定的，那么自定义源集可能是正确的方法。例如，集成测试通常是项目的一部分，因为它们测试中的代码`main`。此外，它们通常具有独立于`test`源集的自己的依赖关系，或者需要与自定义`Test`任务一起运行。

其他常见方案不太明确，可能有更好的解决方案。例如：

- 单独的API和实现JAR-将它们作为单独的项目可能是有意义的，特别是如果您已经具有多项目构建
- 生成的源-如果生成的源应使用生产代码进行编译，则将其路径添加到`main`源集中，并确保`compileJava`任务依赖于生成源的任务

如果不确定是否要创建自定义源集，请继续进行操作。它应该简单明了，如果不是，则可能不是适合该工作的工具。

## [管理资源](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_resources)

许多Java项目都使用源文件以外的资源，例如图像，配置文件和本地化数据。有时，这些文件只需要原封不动地打包，有时需要将它们作为模板文件或以其他方式进行处理。无论哪种方式，Java插件都会为处理其相关资源处理的每个源集添加特定的[复制](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.Copy.html)任务。

任务名称如下的约定-或为源集-它会自动复制任何文件*的src / [sourceSet] /资源*到将被包括在生产JAR的目录。该目标目录也将包含在测试的运行时类路径中。`process*SourceSet*Resources``processResources``main`

由于`processResources`是`Copy`任务的一个实例，因此您可以执行“[使用文件”](https://docs.gradle.org/4.10.2/userguide/working_with_files.html#sec:copying_files)一章中描述的任何处理。

### [Java属性文件和可复制的内部版本](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:properties_files)

您可以通过[WriteProperties](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.WriteProperties.html)任务轻松创建Java属性文件，该任务解决了一个众所周知的问题，`Properties.store()`即降低[增量构建](https://docs.gradle.org/4.10.2/userguide/more_about_tasks.html#sec:up_to_date_checks)的用处。

即使使用相同的属性和值，用于编写属性文件的标准Java API也会每次生成一个唯一的文件，因为注释中包括了时间戳。`WriteProperties`如果所有属性均未更改，则Gradle的任务逐字节生成完全相同的输出。这是通过对属性文件的生成方式进行一些调整来实现的：

- 没有时间戳注释添加到输出
- 行分隔符与系统无关，但是可以显式配置（默认为`'\n'`）
- 属性按字母顺序排序

有时可能需要在不同的计算机上以字节为单位重新创建档案。您要确保从源代码构建工件，无论在何时何地构建，都逐字节产生相同的结果。这对于诸如reproducible-builds.org之类的项目是必需的。

这些调整不仅可以导致更好的增量构建集成，而且还有助于可[复制的构建](https://reproducible-builds.org/)。本质上，可重现的构建可确保您无论在何时何地在什么系统上运行，都可以从构建执行中看到相同的结果，包括测试结果和生产二进制文件。

## [运行测试](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:running_java_tests)

除了在*src / test / java中*提供单元测试的自动编译功能外，Java插件还对运行使用JUnit 3、4和5的测试提供了本机支持（[Gradle 4.6中](https://docs.gradle.org/4.6/release-notes.html#junit-5-support)提供[了对](https://docs.gradle.org/4.6/release-notes.html#junit-5-support)JUnit 5的支持）和TestNG。你得到：

- 使用源集的[Test](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.testing.Test.html)`test`类型的自动任务`test`
- HTML测试报告，其中包含*所有* `Test`运行任务的结果
- 轻松过滤要运行的测试
- 精细控制测试的运行方式
- 有机会创建自己的测试执行和测试报告任务

您*不会*`Test`为声明的每个源集获得任务，因为不是每个源集都代表测试！这就是为什么您通常需要为集成和验收测试之类的东西[创建自己的`Test`任务](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:custom_java_source_sets)，如果它们不包含在`test`源集中。

由于涉及测试的内容很多，因此该主题有其[自己的章节](https://docs.gradle.org/4.10.2/userguide/java_testing.html#java_testing)，我们在其中进行介绍：

- 测试如何运行
- 如何通过过滤运行测试的子集
- Gradle如何发现测试
- 如何配置测试报告并添加自己的报告任务
- 如何利用特定的JUnit和TestNG功能

您还可以在DSL参考中的[Test上](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.testing.Test.html)了解有关配置测试的更多信息。

## [包装出版](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:java_packaging)

How you package and potentially publish your Java project depends on what type of project it is. Libraries, applications, web applications and enterprise applications all have differing requirements. In this section, we will focus on the bare bones provided by the Java Plugin.

The one and only packaging feature provided by the Java Plugin directly is a `jar` task that packages all the compiled production classes and resources into a single JAR. This JAR is then added as an artifact — as opposed to a dependency — in the `archives` configuration, hence why it is automatically built by the `assemble` task.

如果要构建任何其他JAR或替代归档，则必须应用适当的插件或手动创建任务。例如，如果您想要一个生成“源” JAR的`Jar`任务，请像这样定义自己的任务：

### [示例：定义自定义任务以创建“源” JAR](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_defining_a_custom_task_to_create_a_sources_jar)

build.gradle

```groovy
task sourcesJar(type: Jar) {
    classifier = 'sources'
    from sourceSets.main.allJava
}
```

有关可用配置选项的更多详细信息，请参见[Jar](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.bundling.Jar.html)。请注意，您需要使用`classifier`而不是`appendix`此处来正确发布JAR。

如果您想创建一个“超级”（又称“胖”）JAR，则可以使用如下任务定义：

### [示例：创建Java uber或fat JAR](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_creating_a_java_uber_or_fat_jar)

build.gradle

```groovy
plugins {
    id 'java'
}

version = '1.0.0'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'commons-io:commons-io:2.6'
}

task uberJar(type: Jar) {
    appendix = 'uber'

    from sourceSets.main.output
    from configurations.runtimeClasspath.
                                         findAll { it.name.endsWith('jar') }.
                                         collect { zipTree(it) }
}
```

创建JAR后，有多种选择可用于发布它：

- 在[Maven的插件发布](https://docs.gradle.org/4.10.2/userguide/publishing_maven.html#publishing_maven)
- 在[常春藤发布插件](https://docs.gradle.org/4.10.2/userguide/publishing_ivy.html#publishing_ivy)
- 该`uploadArchives`任务-[原发布机制](https://docs.gradle.org/4.10.2/userguide/artifact_management.html#artifact_management)-这既Ivy和工作（如果你申请的[Maven插件](https://docs.gradle.org/4.10.2/userguide/maven_plugin.html#maven_plugin)）的Maven

前两个“发布”插件是首选选项。

### [修改JAR清单](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:jar_manifest)

`Jar`，`War`和`Ear`任务的每个实例都有一个`manifest`属性，该属性使您可以自定义进入相应归档文件的*MANIFEST.MF*文件。下面的示例演示如何在JAR清单中设置属性：

#### [示例：MANIFEST.MF的自定义](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_customization_of_manifest_mf)

build.gradle

```groovy
jar {
    manifest {
        attributes("Implementation-Title": "Gradle",
                   "Implementation-Version": version)
    }
}
```

请参阅[清单](https://docs.gradle.org/4.10.2/javadoc/org/gradle/api/java/archives/Manifest.html)以获取其提供的配置选项。

您还可以创建的独立实例`Manifest`。这样做的原因之一是在JAR之间共享清单信息。下面的示例演示如何在JAR之间共享通用属性：

#### [示例：创建清单对象。](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_creating_a_manifest_object)

build.gradle

```groovy
ext.sharedManifest = manifest {
    attributes("Implementation-Title": "Gradle",
               "Implementation-Version": version)
}
task fooJar(type: Jar) {
    manifest = project.manifest {
        from sharedManifest
    }
}
```

您可以使用的另一种选择是将清单合并到单个`Manifest`对象中。这些源清单可以采用文本形式或其他`Manifest`对象形式。在以下示例中，源清单是除上一个示例中`sharedManifest`的`Manifest`对象之外的所有文本文件：

#### [示例：为特定存档单独的MANIFEST.MF](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_separate_manifest_mf_for_a_particular_archive)

build.gradle

```groovy
task barJar(type: Jar) {
    manifest {
        attributes key1: 'value1'
        from sharedManifest, 'src/config/basemanifest.txt'
        from('src/config/javabasemanifest.txt',
             'src/config/libbasemanifest.txt') {
            eachEntry { details ->
                if (details.baseValue != details.mergeValue) {
                    details.value = baseValue
                }
                if (details.key == 'foo') {
                    details.exclude()
                }
            }
        }
    }
}
```

清单按照`from`语句中声明的顺序合并。如果基本清单和合并清单都为同一个键定义值，则默认情况下，合并清单将获胜。您可以通过添加`eachEntry`操作来完全自定义合并行为，在操作中您可以访问结果清单的每个条目的[ManifestMergeDetails](https://docs.gradle.org/4.10.2/javadoc/org/gradle/api/java/archives/ManifestMergeDetails.html)实例。请注意，合并是懒洋洋地生成JAR时或完成后，无论是在`Manifest.writeTo()`或`Manifest.getEffectiveManifest()`调用。

说到`writeTo()`，您可以使用它随时轻松地将清单写入磁盘，如下所示：

#### [示例：将MANIFEST.MF保存到磁盘](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_saving_a_manifest_mf_to_disk)

build.gradle

```groovy
jar.manifest.writeTo("$buildDir/mymanifest.mf")
```

## [生成API文档](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:generating_javadocs)

Java插件提供了[Javadoc](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.javadoc.Javadoc.html)`javadoc`类型的任务，该任务将为所有生产代码（即，源集中的任何源代码）生成标准Javadocs 。该任务支持[Javadoc参考文档中](http://docs.oracle.com/javase/7/docs/technotes/tools/windows/javadoc.html#options)描述的核心Javadoc和标准doclet选项。有关这些选项的完整列表，请参见[CoreJavadocOptions](https://docs.gradle.org/4.10.2/javadoc/org/gradle/external/javadoc/CoreJavadocOptions.html)和[StandardJavadocDocletOptions](https://docs.gradle.org/4.10.2/javadoc/org/gradle/external/javadoc/StandardJavadocDocletOptions.html)。`main`

作为您可以做的事的一个例子，想象一下您想在Javadoc注释中使用Asciidoc语法。为此，您需要将Asciidoclet添加到Javadoc的doclet路径。这是一个执行此操作的示例：

### [示例：将自定义doclet与Javadoc结合使用](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_using_a_custom_doclet_with_javadoc)

build.gradle

```groovy
configurations {
    asciidoclet
}

dependencies {
    asciidoclet 'org.asciidoctor:asciidoclet:1.+'
}

javadoc {
    options.docletpath = configurations.asciidoclet.files.toList()
    options.doclet = 'org.asciidoctor.Asciidoclet'
}
```

您不必为此创建配置，但这是一种处理独特目的所需依赖项的绝妙方法。

您可能还想创建自己的Javadoc任务，例如为测试生成API文档：

### [示例：定义自定义Javadoc任务](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#example_defining_a_custom_javadoc_task)

build.gradle

```groovy
task testJavadoc(type: Javadoc) {
    source = sourceSets.test.allJava
}
```

这些只是您可能会遇到的两个不重要但常见的自定义。

## [清理构建](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:cleaning_java_build)

Java插件`clean`通过应用[基本插件](https://docs.gradle.org/4.10.2/userguide/base_plugin.html#base_plugin)将任务添加到项目中。此任务只是删除`$buildDir`目录中的所有内容，因此为什么要始终将构建生成的文件放在其中。该任务是[Delete的](https://docs.gradle.org/4.10.2/dsl/org.gradle.api.tasks.Delete.html)一个实例，您可以通过设置其`dir`属性来更改其删除的目录。

## [构建Java库](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_libraries)

库项目的独特之处在于它们被其他Java项目使用（或“消耗”）。这意味着随JAR文件一起发布的依赖项元数据（通常以Maven POM的形式）至关重要。特别是，库的使用者应能够区分两种不同类型的依赖关系：仅依赖于编译库的依赖关系和也依赖于编译使用者的依赖关系。

Gradle通过[Java库插件](https://docs.gradle.org/4.10.2/userguide/java_library_plugin.html#java_library_plugin)来管理这种区别，该[插件](https://docs.gradle.org/4.10.2/userguide/java_library_plugin.html#java_library_plugin)除了本章介绍的*实现*外，还引入了*api*配置。如果依赖项的类型出现在库的公共类的公共字段或方法中，则该依赖项通过库的公共API公开，因此应将其添加到*api*配置中。否则，依赖项是内部实现细节，应将其添加到*Implementation中*。

| ✨    | Java库插件也会自动应用标准Java插件。 |
| ---- | ------------------------------------ |
|      |                                      |

如果不确定API和实现依赖项之间的区别，请参阅[Java库插件一章](https://docs.gradle.org/4.10.2/userguide/java_library_plugin.html#sec:java_library_recognizing_dependencies)中的详细说明。此外，您可以在相应的[*指南中*](https://guides.gradle.org/building-java-libraries/)看到构建Java库的基本，实际示例。

## [构建Java应用程序](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_applications)

无法将打包为JAR的Java应用程序设置为易于从命令行或桌面环境启动。该[应用程序插件](https://docs.gradle.org/4.10.2/userguide/application_plugin.html#application_plugin)通过创建一个分布，其包括生产JAR，它的依赖和启动脚本类Unix和Windows系统解决了命令行方面。

有关更多详细信息，请参见插件的章节，但这是您所获得的快速摘要：

- `assemble` 创建应用程序的ZIP和TAR发行版，其中包含运行它所需的一切
- 一个`run`是开始从构建的应用程序（简单的测试）的任务
- Shell和Windows Batch脚本启动应用程序

请注意，您将需要在构建脚本中显式应用Java插件。

您可以在相应的[*指南中*](https://guides.gradle.org/building-java-applications/)看到构建Java应用程序的基本示例。

## [构建Java Web应用程序](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_webapps)

Java Web应用程序可以根据您使用的技术以多种方式打包和部署。例如，您可以将[Spring Boot](https://projects.spring.io/spring-boot/)与胖JAR或[Netty上](https://netty.io/)运行的基于[Reactive](https://www.reactivemanifesto.org/)的系统一起使用。无论您使用哪种技术，Gradle及其庞大的插件社区都可以满足您的需求。但是，Core Gradle仅直接支持部署为WAR文件的传统基于Servlet的Web应用程序。

该支持来自[War插件](https://docs.gradle.org/4.10.2/userguide/war_plugin.html#war_plugin)，该[插件](https://docs.gradle.org/4.10.2/userguide/war_plugin.html#war_plugin)会自动应用Java插件并添加一个额外的打包步骤，该步骤执行以下操作：

- 将静态资源从*src / main / webapp*复制到WAR的根目录中
- 将编译后的生产类复制到WAR的*WEB-INF / classes*子目录中
- 将库依赖项复制到WAR的*WEB-INF / lib*子目录中

这是由`war`任务完成的，它有效地替换了`jar`任务（尽管该任务仍然存在）并附加到`assemble`生命周期任务上。有关更多详细信息和配置选项，请参见插件的章节。

没有直接从内部版本运行Web应用程序的核心支持，但是我们建议您尝试使用[Gretty](https://plugins.gradle.org/plugin/org.gretty)社区插件，该插件提供了嵌入式Servlet容器。

## [构建Java EE应用程序](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#sec:building_java_enterprise_apps)

多年来，Java企业系统已经发生了很大的变化，但是如果您仍要部署到JEE应用服务器，则可以使用[Ear Plugin](https://docs.gradle.org/4.10.2/userguide/ear_plugin.html#ear_plugin)。这增加了约定和构建EAR文件的任务。插件的章节有更多详细信息。

------

[1](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#_footnoteref_1)。实际上，添加到`archives`配置中的任何工件都将由`assemble`

[2](https://docs.gradle.org/4.10.2/userguide/building_java_projects.html#_footnoteref_2)。有关更多详细信息，`gradle.properties`请参见[Gradle配置属性。](https://docs.gradle.org/4.10.2/userguide/build_environment.html#sec:gradle_configuration_properties)



