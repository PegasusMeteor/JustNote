# CheckStyle 整理

## 现装介绍

目前 使用的CheckStyle 版本是checkstyle-6.16.1-all.jar。

目前的CheckStyle的验证方式是 通过 git hook 的方式来进行，验证。即，在push 代码的时候，通过一段shell 脚本来验证 本次变更中 哪些Java代码是 不合规范的。

**Each module has a severity property that a Checkstyle audit assigns to all violations of the check. The default severity level of a check is error.**

## 旧的CheckStyle 梳理

```xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE module PUBLIC "-//Puppy Crawl//DTD Check Configuration 1.3//EN" "http://www.puppycrawl.com/dtds/configuration_1_3.dtd">
 
<!--
    Checkstyle-Configuration: easyop-cs-2.0
    Description: easyop checkstyle rule 2.0, since 2018-02-01
-->
<module name="Checker">
    <property name="severity" value="error"/>
    <module name="TreeWalker">
        <!-- 当配置为 TreeWalker 子模块时，保存当前文件内容以供全局访问。例如，过滤器可以通过此模块访问当前文件内容。 -->
        <module name="FileContentsHolder"/>
        <!-- 参数分配通常被认为是糟糕的编程实践。强制开发人员将参数声明为 final 通常是繁重的。检查确保永远不会分配参数将两全其美。 -->
        <module name="ParameterAssignment">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 不许使用未被简化的条件表达式 -->
        <module name="SimplifyBooleanExpression">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 检查静态的非final变量名是否符合指定的模式，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="StaticVariableName">
            <property name="severity" value="error"/>
        </module>
        <!-- 检查方法名称是否符合指定的模式，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="MethodName">
            <property name="severity" value="error"/>
        </module>
        <!-- 不许内部赋值 String s = Integer.toString(i = 2) -->
        <module name="InnerAssignment">
            <property name="severity" value="error"/>
        </module>
        <!-- 检查未使用的导入包 -->
        <module name="UnusedImports">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 检查字符串字面值是否与==或!=一起使用。因为==将比较对象引用，而不是字符串的实际值，所以应该使用String.equals() -->
        <module name="StringLiteralEquality">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 检查接口和annotation中是否有多余修饰符，如接口方法不必使用public -->
        <module name="RedundantModifier">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 检查指定的类型是否声明为要抛出。声明一个方法会引发java.lang.Error或java.lang.RuntimeException几乎是不可接受的 -->
        <module name="IllegalThrows">
            <property name="severity" value="error"/>
        </module>
        <!--检查长匿名内部类长度，最大30  -->
        <module name="AnonInnerLength">
            <property name="severity" value="error"/>
            <property name="max" value="30"/>
        </module>
        <!-- 检查实例变量名称是否符合指定的模式，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="MemberName">
            <property name="severity" value="error"/>
        </module>
        <!-- 检查常量名称是否符合指定的模式，默认"^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$" -->
        <module name="ConstantName">
            <property name="severity" value="error"/>
        </module>
        <!-- 包名的检查（只允许小写字母），默认^[a-z]+(\.[a-zA-Z_][a-zA-Z_0-9_]*)*$ -->
        <module name="PackageName">
            <property name="severity" value="error"/>
        </module>
        <!-- 检查default是否在switch语句中的所有case之后。 -->
        <module name="DefaultComesLast">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 检查某一个类中equals() 和 hashCode() 方法是否同时被重写，即一个被重写，另外一个也必须被重写。通常将二者看成一个override 操作。 -->
        <module name="EqualsHashCode">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!--不许使用未被简化的布尔返回值

            if (valid()){
                return false;
            }else{
                return true;
            }  -->
        <module name="SimplifyBooleanReturn">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 检查局部变量或参数是否不影响同一类中定义的字段。 -->
        <module name="HiddenField">
            <property name="severity" value="error"/>
            <property name="tokens" value="VARIABLE_DEF"/>
            <property name="ignoreConstructorParameter" value="true"/>
            <property name="ignoreSetter" value="true"/>
        </module>
        <!-- 检查局部final变量名称是否符合指定的模式。try语句中的catch参数和资源被认为是局部的、最终的变量，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="LocalFinalVariableName">
            <property name="severity" value="error"/>
        </module>
        <!-- 函数的分支复杂度，不超过10 -->
        <module name="CyclomaticComplexity">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 检查类成员的可见性。只有static final、不可变或由指定注释成员注释的才可以是public;除非设置了protectedAllowed或packageAllowed属性，否则其他类成员必须是私有的。  -->
        <module name="VisibilityModifier">
            <property name="severity" value="warning"/>
            <property name="packageAllowed" value="true"/>
            <property name="protectedAllowed" value="true"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="inherit"/>
        </module>
        <!-- 检查局部的非final变量名是否符合指定的模式。catch参数被认为是一个局部变量，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="LocalVariableName">
            <property name="severity" value="error"/>
            <property name="format" value="^[a-z][_a-zA-Z0-9]*$"/>
        </module>
        <!-- 检查方法参数名称是否符合指定的模式，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="ParameterName">
            <property name="severity" value="error"/>
        </module>
        <!-- 布尔表达式的复杂度，不超过5 -->
        <module name="BooleanExpressionComplexity">
            <property name="severity" value="error"/>
            <property name="max" value="5"/>
        </module>
        <!-- 检查长方法和构造函数，默认最大150行。不统计空行。  -->
        <module name="MethodLength">
            <property name="severity" value="error"/>
            <property name="countEmpty" value="false"/>
        </module>
        <!-- 将嵌套的if-else块限制到指定的深度  -->
        <module name="NestedIfDepth">
            <property name="severity" value="info"/>
            <property name="max" value="5"/>
        </module>
        <!-- 将嵌套的try-catch-finally块限制到指定的深度。默认1 -->
        <module name="NestedTryDepth">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 方法的参数个数，默认为7。 -->
        <module name="ParameterNumber">
            <property name="severity" value="error"/>
        </module>
        <!-- 查找嵌套块（在代码中自由使用的块）。 -->
        <module name="AvoidNestedBlocks">
            <property name="severity" value="error"/>
        </module>
        <!-- 检查字符串字面值的任何组合是否位于equals()比较的左侧。还检查分配给某些字段的字符串字面量(例如someString。= (anotherString =“文本”)  -->
        <module name="EqualsAvoidNull">
            <property name="severity" value="error"/>
        </module>
        <!-- 检查优先使用工厂方法的非法实例化。 根据项目的不同，对于某些类，最好通过工厂方法创建实例而不是调用构造函数。 -->
        <module name="IllegalInstantiation">
            <property name="severity" value="error"/>
        </module>
        <!-- 检查 for 循环控制变量在 for 块内没有被修改 -->
        <module name="ModifiedControlVariable">
            <property name="severity" value="error"/>
        </module>
        <!-- Class或Interface名检查，默认^[A-Z][a-zA-Z0-9]*$  -->
        <module name="TypeName"/>
        <!-- 检查数组类型定义的样式 -->
        <module name="ArrayTypeStyle"/>
        <!-- 检查long型定义是否有大写的“L” -->
        <module name="UpperEll"/>
        <!-- 检查所有的interface和class allowUnknownTags：控制在无法识别 Javadoc 标记时是否忽略违规。 -->
        <module name="JavadocType">
            <property name="allowUnknownTags" value="true"/>
        </module>
        <!-- 多余的导入语句 -->
        <module name="RedundantImport"/>
        <!--  检查空块。此检查不验证顺序块。 -->
        <module name="EmptyBlock">
            <property name="severity" value="error"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="error"/>
        </module>
        <!-- 检查每个变量声明是否在其自己的语句中并在其自己的行上。 -->
        <module name="MultipleVariableDeclarations"/>
        <!-- 限制方法、构造函数和 lambda 表达式中返回语句的数量。 -->
        <module name="ReturnCount">
            <property name="max" value="4"/>
        </module>
        <!-- 验证 Javadoc 注释以帮助确保它们的格式正确。 -->
        <module name="JavadocStyle">
            <property name="severity" value="ignore"/>
            <property name="checkFirstSentence" value="false"/>
            <metadata name="net.sf.eclipsecs.core.lastEnabledSeverity" value="inherit"/>
        </module>
    </module>
    <!-- 文件长度不超过1500行 -->
    <module name="FileLength">
        <property name="severity" value="error"/>
    </module>
</module>
```

## CheckStyle 升级


maven 配置。

```xml
<dependencies>
    <dependency>
        <groupId>com.puppycrawl.tools</groupId>
        <artifactId>checkstyle</artifactId>
        <version>8.45.1</version>
    </dependency>
</dependencies>
<build>
    <!-- 插件依赖管理 -->
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>3.1.2</version>
                <dependencies>
                    <dependency>
                        <groupId>com.puppycrawl.tools</groupId>
                        <artifactId>checkstyle</artifactId>
                        <version>8.45</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <configLocation>checkstyle.xml</configLocation>
                    <encoding>UTF-8</encoding>
                    <consoleOutput>true</consoleOutput>
                    <failsOnError>true</failsOnError>
                    <!-- 警告信息也直接失败 -->
                    <violationSeverity>error</violationSeverity>
                    <linkXRef>false</linkXRef>
                    <outputFile>target/reports/checkstyle/checkstyle-result.xml</outputFile>
                    <outputDirectory>target/reports/checkstyle</outputDirectory>
                    <outputFileFormat>xml</outputFileFormat>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <!-- verify 阶段 -->
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-checkstyle-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.7</version>
            <executions>
                <execution>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <!-- attached to Maven test phase -->
                <execution>
                    <id>report</id>
                    <phase>test</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>

```

## CheckStyle 升级

```xml
<?xml version="1.0"?>

<!DOCTYPE module PUBLIC "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN" "https://checkstyle.org/dtds/configuration_1_3.dtd">

<!--
This is a checkstyle configuration file. For descriptions of
what the following rules do, please see the checkstyle configuration
page at http://checkstyle.sourceforge.net/config.html.
This file is based on the checkstyle file of Apache Beam.
-->

<module name="Checker">
    <module name="FileLength">
        <property name="max" value="1500" />
    </module>
    <!--   单行代码长度 -->
    <module name="LineLength">
        <!-- Checks if a line is too long. -->
        <property name="max" value="150" />
        <property name="ignorePattern" value="^(package .*;\s*)|(import .*;\s*)|( *\* .*https?://.*)$" />
    </module>

    <!-- All Java AST specific tests live under TreeWalker module. -->
    <module name="TreeWalker">
        <!-- 参数分配通常被认为是糟糕的编程实践。强制开发人员将参数声明为 final 通常是繁重的。检查确保永远不会分配参数将两全其美。 -->
        <module name="ParameterAssignment" />

        <!-- 不许使用未被简化的条件表达式 -->
        <module name="SimplifyBooleanExpression" />

        <!-- 检查静态的非final变量名是否符合指定的模式，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="StaticVariableName" />

        <!-- 检查方法名称是否符合指定的模式，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="MethodName" />
        <!-- 不许内部赋值 String s = Integer.toString(i = 2) -->
        <module name="InnerAssignment" />

        <!-- 检查未使用的导入包 -->
        <module name="UnusedImports">
            <property name="processJavadoc" value="true" />
            <message key="import.unused" value="Unused import: {0}." />
        </module>
        <!-- 检查字符串字面值是否与==或!=一起使用。因为==将比较对象引用，而不是字符串的实际值，所以应该使用String.equals() -->
        <module name="StringLiteralEquality" />

        <!-- 检查接口和annotation中是否有多余修饰符，如接口方法不必使用public -->
        <module name="RedundantModifier">
            <property name="tokens" value="VARIABLE_DEF, ANNOTATION_FIELD_DEF, INTERFACE_DEF, CLASS_DEF, ENUM_DEF" />
        </module>
        <!-- 检查指定的类型是否声明为要抛出。声明一个方法会引发java.lang.Error或java.lang.RuntimeException几乎是不可接受的 -->
        <module name="IllegalThrows" />

        <!--检查长匿名内部类长度，最大30  -->
        <module name="AnonInnerLength">
            <property name="max" value="30" />
        </module>
        <!-- 检查实例变量名称是否符合指定的模式，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="MemberName" />

        <!-- 检查常量名称是否符合指定的模式，默认"^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$" -->
        <module name="ConstantName" />
        <!-- 包名的检查（只允许小写字母），默认^[a-z]+(\.[a-zA-Z_][a-zA-Z_0-9_]*)*$ -->
        <module name="PackageName">
            <property name="format" value="^[a-z]+(\.[a-z][a-z0-9]{1,})*$" />
        </module>
        <!-- 检查default是否在switch语句中的所有case之后。 -->
        <module name="DefaultComesLast" />

        <!-- 检查某一个类中equals() 和 hashCode() 方法是否同时被重写，即一个被重写，另外一个也必须被重写。通常将二者看成一个override 操作。 -->
        <module name="EqualsHashCode" />

        <!--不许使用未被简化的布尔返回值

            if (valid()){
                return false;
            }else{
                return true;
            }  -->
        <module name="SimplifyBooleanReturn" />

        <!-- 检查局部变量或参数是否不影响同一类中定义的字段。 -->
        <module name="HiddenField">
            <property name="tokens" value="VARIABLE_DEF" />
            <property name="ignoreConstructorParameter" value="true" />
            <property name="ignoreSetter" value="true" />
        </module>
        <!-- 检查局部final变量名称是否符合指定的模式。try语句中的catch参数和资源被认为是局部的、最终的变量，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="LocalFinalVariableName" />
        <!-- 函数的分支复杂度，不超过10 -->
        <module name="CyclomaticComplexity" />
        <!-- 检查类成员的可见性。只有static final、不可变或由指定注释成员注释的才可以是public;除非设置了protectedAllowed或packageAllowed属性，否则其他类成员必须是私有的。  -->
        <module name="VisibilityModifier">
            <property name="severity" value="warning" />
            <property name="packageAllowed" value="true" />
            <property name="protectedAllowed" value="true" />
        </module>
        <!-- 检查局部的非final变量名是否符合指定的模式。catch参数被认为是一个局部变量，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="LocalVariableName">
            <property name="format" value="^[a-z][_a-zA-Z0-9]*$" />
        </module>
        <!-- 检查方法参数名称是否符合指定的模式，默认"^[a-z][a-zA-Z0-9]*$" -->
        <module name="ParameterName" />
        <!-- 布尔表达式的复杂度，不超过5 -->
        <module name="BooleanExpressionComplexity">
            <property name="max" value="5" />
        </module>
        <!-- 检查长方法和构造函数，默认最大150行。不统计空行。  -->
        <module name="MethodLength">
            <property name="countEmpty" value="false" />
        </module>
        <!-- 将嵌套的if-else块限制到指定的深度  -->
        <module name="NestedIfDepth">
            <property name="severity" value="info" />
            <property name="max" value="5" />
        </module>
        <!-- 将嵌套的try-catch-finally块限制到指定的深度。默认1 -->
        <module name="NestedTryDepth" />

        <!-- 方法的参数个数，默认为7。 -->
        <module name="ParameterNumber" />
        <!-- 查找嵌套块（在代码中自由使用的块）。 -->
        <module name="AvoidNestedBlocks" />
        <!-- 检查字符串字面值的任何组合是否位于equals()比较的左侧。还检查分配给某些字段的字符串字面量(例如someString。= (anotherString =“文本”)  -->
        <module name="EqualsAvoidNull" />

        <!-- 检查优先使用工厂方法的非法实例化。 根据项目的不同，对于某些类，最好通过工厂方法创建实例而不是调用构造函数。 -->
        <module name="IllegalInstantiation" />
        <!-- 检查 for 循环控制变量在 for 块内没有被修改 -->
        <module name="ModifiedControlVariable" />
        <!-- Class或Interface名检查，默认^[A-Z][a-zA-Z0-9]*$  -->
        <module name="TypeName" />
        <!-- 检查数组类型定义的样式 -->
        <module name="ArrayTypeStyle" />
        <!-- 检查long型定义是否有大写的“L” -->
        <module name="UpperEll" />
        <!-- 检查所有的interface和class allowUnknownTags：控制在无法识别 Javadoc 标记时是否忽略违规。 -->
        <module name="JavadocType">
            <property name="allowUnknownTags" value="true" />
        </module>
        <!-- 多余的导入语句 -->
        <module name="RedundantImport">
            <!-- Checks for redundant import statements. -->
            <message key="import.redundancy" value="Redundant import {0}." />
        </module>
        <!--  检查空块。此检查不验证顺序块。 -->
        <module name="EmptyBlock" />

        <!-- 检查每个变量声明是否在其自己的语句中并在其自己的行上。 -->
        <module name="MultipleVariableDeclarations" />
        <!-- 限制方法、构造函数和 lambda 表达式中返回语句的数量。 -->
        <module name="ReturnCount">
            <property name="max" value="4" />
        </module>
        <!-- 验证 Javadoc 注释以帮助确保它们的格式正确。 -->
        <module name="JavadocStyle">
            <property name="severity" value="ignore" />
            <property name="checkFirstSentence" value="false" />
        </module>


        <!--
        FLINK CUSTOM CHECKS
        -->
        <!-- 这个是换行使用 tab 缩进       -->
        <module name="RegexpSinglelineJava">
            <property name="format" value="^\t* +\t*\S" />
            <property name="message" value="Line has leading space characters; indentation should be performed with tabs only." />
            <property name="ignoreComments" value="true" />
            <!--  每个文件最多出现100个.相当于先屏蔽掉  -->
            <property name="maximum" value="100" />
        </module>
        <!-- 检查是否有多个分号出现 (e.g. ";;"). -->
        <module name="RegexpSinglelineJava">
            <property name="format" value=";{2,}" />
            <property name="message" value="Use one semicolon" />
            <property name="ignoreComments" value="true" />
        </module>
        <!-- Prohibit T.getT() methods for standard boxed types -->
        <module name="Regexp">
            <property name="format" value="Boolean\.getBoolean" />
            <property name="illegalPattern" value="true" />
            <property name="message" value="Use System.getProperties() to get system properties." />
        </module>

        <module name="Regexp">
            <property name="format" value="Integer\.getInteger" />
            <property name="illegalPattern" value="true" />
            <property name="message" value="Use System.getProperties() to get system properties." />
        </module>

        <module name="Regexp">
            <property name="format" value="Long\.getLong" />
            <property name="illegalPattern" value="true" />
            <property name="message" value="Use System.getProperties() to get system properties." />
        </module>

        <!--
        IllegalImport cannot blacklist classes so we have to fall back to Regexp.
        下面这些包名，或者类名，已经被加入黑名单，禁止使用。
        -->

        <!-- forbid use of commons lang validate -->
        <module name="Regexp">
            <property name="format" value="org\.apache\.commons\.lang3\.Validate" />
            <property name="illegalPattern" value="true" />
            <property name="message" value="Use Guava Checks instead of Commons Validate. Please refer to the coding guidelines." />
        </module>
        <module name="Regexp">
            <property name="format" value="org\.apache\.commons\.lang\." />
            <property name="illegalPattern" value="true" />
            <property name="message" value="Use commons-lang3 instead of commons-lang." />
        </module>
        <module name="Regexp">
            <property name="format" value="org\.codehaus\.jettison" />
            <property name="illegalPattern" value="true" />
            <property name="message" value="Use com.fasterxml.jackson instead of jettison." />
        </module>

        <module name="TodoComment" />
        <module name="AvoidStarImport">
            <property name="severity" value="warning" />
        </module>

        <!--   禁止使用的开发包,被加入黑名单的包     -->
        <module name="IllegalImport">
            <property name="illegalPkgs" value="autovalue.shaded, avro.shaded, com.google.api.client.repackaged, com.google.appengine.repackaged, org.codehaus.jackson, io.netty, org.objectweb.asm, com.google.common" />
        </module>


        <!--
        JAVADOC CHECKS
        -->
        <!-- Checks for Javadoc comments.                     -->
        <!-- See http://checkstyle.sf.net/config_javadoc.html -->
        <module name="JavadocMethod">
            <property name="allowMissingParamTags" value="true" />
            <property name="allowMissingReturnTag" value="true" />
        </module>

        <module name="JavadocType">
            <property name="scope" value="protected" />
            <property name="allowMissingParamTags" value="true" />
        </module>


        <!--
        NAMING CHECKS
        -->

        <!-- Validates static, final fields against the expression "^[A-Z][a-zA-Z0-9]*$". -->
        <module name="TypeNameCheck">
            <metadata name="altname" value="TypeName" />
        </module>
        <!--    常量名称检测    -->
        <module name="ConstantNameCheck">
            <!-- Validates non-private, static, final fields against the supplied
            public/package final fields "^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$". -->
            <metadata name="altname" value="ConstantName" />
            <property name="applyToPublic" value="true" />
            <property name="applyToProtected" value="true" />
            <property name="applyToPackage" value="true" />
            <property name="applyToPrivate" value="false" />
            <property name="format" value="^([A-Z][A-Z0-9]*(_[A-Z0-9]+)*|FLAG_.*)$" />
            <message key="name.invalidPattern" value="Variable ''{0}'' should be in ALL_CAPS (if it is a constant) or be private (otherwise)." />
        </module>

        <module name="StaticVariableNameCheck">
            <!-- Validates static, non-final fields against the supplied
            expression "^[a-z][a-zA-Z0-9]*_?$". -->
            <metadata name="altname" value="StaticVariableName" />
            <property name="applyToPublic" value="true" />
            <property name="applyToProtected" value="true" />
            <property name="applyToPackage" value="true" />
            <property name="applyToPrivate" value="true" />
            <property name="format" value="^[a-z][a-zA-Z0-9]*_?$" />
        </module>

        <module name="MemberNameCheck">
            <!-- Validates non-static members against the supplied expression. -->
            <metadata name="altname" value="MemberName" />
            <property name="applyToPublic" value="true" />
            <property name="applyToProtected" value="true" />
            <property name="applyToPackage" value="true" />
            <property name="applyToPrivate" value="true" />
            <property name="format" value="^[a-z][a-zA-Z0-9]*$" />
        </module>

        <module name="MethodNameCheck">
            <!-- Validates identifiers for method names. -->
            <metadata name="altname" value="MethodName" />
            <property name="format" value="^[a-z][a-zA-Z0-9]*(_[a-zA-Z0-9]+)*$" />
        </module>


        <!--
        LENGTH and CODING CHECKS
        -->

        <!--  检查代码块的左花括号 ('{') 的位置-->
        <module name="LeftCurly" />

        <!--   检查代码块的右花括号 ('}') 的位置     -->
        <module name="RightCurly">
            <!-- Checks right curlies on CATCH, ELSE, and TRY blocks are on
            the same line. e.g., the following example is fine:
            <pre>
            if {
            ...
            } else
            </pre>
            -->
            <!-- This next example is not fine:
            <pre>
            if {
            ...
            }
            else
            </pre>
            -->
            <property name="option" value="same" />
        </module>

        <!-- 检查代码块周围的大括号。 -->
        <module name="NeedBraces">
            <property name="tokens" value="LITERAL_IF, LITERAL_ELSE, LITERAL_FOR, LITERAL_WHILE, LITERAL_DO" />
        </module>

        <!--      检查 switch 语句中的失败。查找 case 包含 Java 代码但缺少 break、return、throw 或 continue 语句的位置-->

        <module name="FallThrough">
            <property name="reliefPattern" value="fall through|Fall through|fallthru|Fallthru|falls through|Falls through|fallthrough|Fallthrough|No break|NO break|no break|continue on" />
        </module>

        <!-- Detects empty statements (standalone ";" semicolon). -->
        <module name="EmptyStatement" />

        <!--
        MODIFIERS CHECKS
        -->

        <!--   检查修饰符的顺序，是否符合 java 标准
        public
        protected
        private
        abstract
        default
        static
        sealed
        non-sealed
        final
        transient
        volatile
        synchronized
        native
        strictfp
        -->
        <module name="ModifierOrder" />


        <!--
        WHITESPACE CHECKS
        -->
        <!-- Checks for empty line separator between tokens. The only
                     excluded token is VARIABLE_DEF, allowing class fields to
                     be declared on consecutive lines.
                -->
        <module name="EmptyLineSeparator">

            <property name="allowMultipleEmptyLines" value="false" />
            <property name="allowMultipleEmptyLinesInsideClassMembers" value="false" />
            <property name="tokens" value="PACKAGE_DEF, IMPORT, CLASS_DEF,
        INTERFACE_DEF, ENUM_DEF, STATIC_INIT, INSTANCE_INIT, METHOD_DEF,
        CTOR_DEF" />
        </module>

        <!-- Checks that various tokens are surrounded by whitespace.
            This includes most binary operators and keywords followed
            by regular or curly braces.
       -->
        <module name="WhitespaceAround">
            <property name="tokens" value="ASSIGN, BAND, BAND_ASSIGN, BOR,
        BOR_ASSIGN, BSR, BSR_ASSIGN, BXOR, BXOR_ASSIGN, COLON, DIV, DIV_ASSIGN,
        EQUAL, GE, GT, LAND, LE, LITERAL_CATCH, LITERAL_DO, LITERAL_ELSE,
        LITERAL_FINALLY, LITERAL_FOR, LITERAL_IF, LITERAL_RETURN,
        LITERAL_SYNCHRONIZED, LITERAL_TRY, LITERAL_WHILE, LOR, LT, MINUS,
        MINUS_ASSIGN, MOD, MOD_ASSIGN, NOT_EQUAL, PLUS, PLUS_ASSIGN, QUESTION,
        SL, SL_ASSIGN, SR_ASSIGN, STAR, STAR_ASSIGN" />

        </module>

        <!-- Checks that commas, semicolons and typecasts are followed by
               whitespace.
          -->
        <module name="WhitespaceAfter">
            <property name="tokens" value="COMMA, SEMI, TYPECAST" />
        </module>
        <!-- Checks that there is no whitespace after various unary operators.
              Linebreaks are allowed.
         -->
        <module name="NoWhitespaceAfter">
            <property name="tokens" value="BNOT, DEC, DOT, INC, LNOT, UNARY_MINUS,UNARY_PLUS" />
            <property name="allowLineBreaks" value="true" />
        </module>

        <!-- Checks that there is no whitespace before various unary operators.
               Linebreaks are allowed.
          -->
        <module name="NoWhitespaceBefore">
            <property name="tokens" value="SEMI, DOT, POST_DEC, POST_INC" />
            <property name="allowLineBreaks" value="true" />
        </module>

        <!-- Checks that operators like + and ? appear at newlines rather than
        at the end of the previous line.
        -->
        <module name="OperatorWrap">
            <property name="option" value="NL" />
            <property name="tokens" value="BAND, BOR, BSR, BXOR, DIV, EQUAL,
            GE, GT, LAND, LE, LITERAL_INSTANCEOF, LOR, LT, MINUS, MOD,
            NOT_EQUAL, PLUS, QUESTION, SL, SR, STAR " />
        </module>

        <!-- Checks that assignment operators are at the end of the line. -->
        <module name="OperatorWrap">
            <property name="option" value="eol" />
            <property name="tokens" value="ASSIGN" />
        </module>

        <!--        检查有关括号填充的策略-->
        <!-- Checks that there is no whitespace before close parens or after
          open parens. -->
        <module name="ParenPad" />
    </module>
</module>
```