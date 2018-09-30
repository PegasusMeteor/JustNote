# dockerfile 

## 镜像的相关操作 

![镜像的相关操作](http://ot2trm1s2.bkt.clouddn.com/docker/imge-operation.png)






没有找到最新版本的dockerfile文档，所以这里最好对比一下官方的dockerfile文档
https://docs.docker.com/engine/reference/builder/



# Dockerfile reference

Docker 能供从`Dockerfile`读取指令来自动构建镜像. `Dockerfile`是一个文本文档，其中包含用户可以在命令行上调用以组合图像的所有命令. 使用 `docker build` 命令，用户可以创建一个连续执行多个命令行指令的自动构建任务。

## Usage

`docker build` 命令根据 `Dockerfile`文件和 和 上下文( *context*) 来构建镜像. 构建上下文，指的是在特定位置，例如 `PATH` or `URL`目录下一系列的文件.  `PATH` 指的是物理文件系统上的一个位置 ,  `URL` 是一个 Git repository 地址.

上下文会被递归进行处理. 所以, `PATH`包括所有子目录 , `URL` 包括存储库及其子模块. 此示例显示了使用当前目录作为上下文的构建命令:

    $ docker build .
    Sending build context to Docker daemon  6.51 MB
    ...

构建由Docker守护程序运行，而不是由CLI运行.构建过程的第一件事是将整个上下文（递归地）发送到守护进程.  在大多数情况下，最好从空目录开始作为上下文，并将Dockerfile保存在该目录中。仅添加构建Dockerfile所需的文件。

>**警告**: 不要使用根目录, `/`, 作为 `PATH` ，因为它可能会导致将硬盘驱动中所有的文件全部发送给Docker daemon。

要在构建上下文中使用文件, `Dockerfile`会引用在指令中指定的文件, 例如,  a `COPY` 指令. 要提高构建的性能, 可以在 `.dockerignore` 文件中指定上下文目录中不需要的文件或者目录。
通常来说,  `Dockerfile` 文件被调用时位于上下文目录的根中。也就是`Dockerfile`文件一定位于构建目录的根环境下. 使用`docker build`命令构建镜像时可以使用 `-f` 来指定文件系统中其他位置上的文件。

    $ docker build -f /path/to/a/Dockerfile .

如果构建成功，我们可以指定 repository 和 tag 来存储新的镜像:

    $ docker build -t shykes/myapp .

要在构建后将镜像标记为多个repositories,可以在使用`build`命令指定多个 `-t` 参数:

    $ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .

在 Docker daemon执行`Dockerfile`中的指令是, 它会初步验证 `Dockerfile` 并在语法不正确时返回错误:

    $ docker build -t test/myapp .
    Sending build context to Docker daemon 2.048 kB
    Error response from daemon: Unknown instruction: RUNCMD

 Docker daemon 会逐个执行 `Dockerfile` 中的指令,
在必要时将每条指令的结果提交给新镜像，最后输出新镜像的ID. Docker daemon 会自动清理我们发送的上下文.

请注意，每条指令都是独立运行的，并且会导致创建新镜像 - 因此`RUN cd /tmp`对下一条指令不会产生任何影响。

只要有可能，Dcoker就会重用中间镜像(缓存),这样会明显加快`docker build`的过程。这会在控制台输出的 `Using cache` 信息中进行提示。


    $ docker build -t svendowideit/ambassador .
    Sending build context to Docker daemon 15.36 kB
    Step 1/4 : FROM alpine:3.2
     ---> 31f630c65071
    Step 2/4 : MAINTAINER SvenDowideit@home.org.au
     ---> Using cache
     ---> 2a1c91448f5f
    Step 3/4 : RUN apk update &&      apk add socat &&        rm -r /var/cache/
     ---> Using cache
     ---> 21ed6e7fbb73
    Step 4/4 : CMD env | grep _TCP= | (sed 's/.*_PORT_\([0-9]*\)_TCP=tcp:\/\/\(.*\):\(.*\)/socat -t 100000000 TCP4-LISTEN:\1,fork,reuseaddr TCP4:\2:\3 \&/' && echo wait) | sh
     ---> Using cache
     ---> 7ea8aef582cc
    Successfully built 7ea8aef582cc

构建缓存仅从那些具有本地父链的镜像中使用。这意味这些镜像是被前面的构建过程中创建的，或者整个镜像链都被`docker load` 命令加载了。如果想要从一个指定的镜像来构建缓存的话，可以通过`--cache-from` 选项来指定。 `--cache-from` 选项指定的镜像可以没有父链，也可以从其他的registries 中pull 下来。



## Format

下面是 `Dockerfile`的格式定义:

```Dockerfile
# Comment
INSTRUCTION arguments
```

指令不区分大小写，但是通常写作大写。这样便于将他们与参数区分开来。


Docker 按顺序执行 `Dockerfile` 中的指令.  `Dockerfile` 文件 **必须以 \`FROM\` 指令开头**.  `FROM` 指令指定了我们构建镜像时依赖的基础镜像 (*Base Image*) . `FROM` 之前仅允许一个或者多个`ARG`指令。`ARG`指令中定义了 `FROM` 命令在执行时需要的参数。.

Docker  `#` 开头的行解释为注释行, 除非这一行是有效的 [parser directive](#parser-directives).一行中其他位置的 `#` 标记都会被当作参数来对待。 例如下面这样的表示:

```Dockerfile
# Comment
RUN echo 'we are running some # of cool things'
```


## Parser directives

解析器指令是可选的，并且会影响处理`Dockerfile`中后续行的方式。解析器指令不会添加新的层到编译过程中，而且也不会显示在编译过程中。 解析器指令是一种特殊的注释格式 `# directive=value`. 一条解析器指令只能被使用一次。

一旦处理了注释，空行或构建指令，Docker就不再查找解析器指令。相反，它将格式为解析器指令的任何内容视为注释，并且不会尝试验证它是否可能是解析器指令。 因此,所有的解析器指令必须在`Dockerfile`文件的最top.

`Parser directives` 不区分大小写 . 但是惯例写成小写. 约定还包括解析器指令后面的空行，解析器指令中不支持行延续符:

根据这些规则，下面的例子是不正确的:

使用行延续符无效:

```Dockerfile
# direc \
tive=value
```

出现两次无效:

```Dockerfile
# directive=value1
# directive=value2

FROM ImageName
```

出现在一个构建指令之后，被当作注释来对待:

```Dockerfile
FROM ImageName
# directive=value
```

出现在一个非解析器指令的注释行之后，也会被当作注释来对待:

```Dockerfile
# About my dockerfile
# directive=value
FROM ImageName
```

系统不认识的解析器指令会被当做注释来对待，而这条指令之后的正确的解析器指令也会被当成注释来对待了。

```Dockerfile
# unknowndirective=value
# knowndirective=value
```

解析器指令中允许使用非换行空格。因此，以下行都被相同地处理:

```Dockerfile
#directive=value
# directive =value
#	directive= value
# directive = value
#	  dIrEcTiVe=value
```

支持如下的一些解析器指令:

* `escape`

## escape

    # escape=\ (backslash)

或者

    # escape=` (backtick)

 `escape` 指定了`Dockerfile`中的转义字符. 如果没有指定，默认的转义字符是 `\`.

转义字符既用于转义行中的字符，也用于转义换行符。 这允许 `Dockerfile` 指令跨越多行. 请注意，无论转义解析器指令是否包含在`Dockerfile`中，*都不会在RUN命令中执行转义，除非在行尾*。

将转义字符设置为`` ` ``在Windows上特别有用，其中`\`是目录路径分隔符。 `` ` ``与Windows PowerShell一致。

下面的例子在`Windows`可能会失败. 第二行行尾的第二个 `\`  会被当做一个新行的转义符, 而不是第一个 `\`作为转义符时的转义对象.类似的, 第三行行尾的 `\`也可能被当做一个转义符 .这个 `Dockerfile`的结果就是第二行和第三行被当做了单独的指令。

```Dockerfile
FROM microsoft/nanoserver
COPY testfile.txt c:\\
RUN dir c:\
```

Results in:

    PS C:\John> docker build -t cmd .
    Sending build context to Docker daemon 3.072 kB
    Step 1/2 : FROM microsoft/nanoserver
     ---> 22738ff49c6d
    Step 2/2 : COPY testfile.txt c:\RUN dir c:
    GetFileAttributesEx c:RUN: The system cannot find the file specified.
    PS C:\John>

上面问题的解决方式之一就是使用`/`作为 `COPY`和`dir`的指令符。但是，这种语法充其量是令人困惑的，因为它对于Windows上的路径来说并不自然，并且最坏的情况是容易出错，因为Windows上的所有命令都不支持/作为路径分隔符。

通过添加 `escape` 解析指令,  `Dockerfile` 就可以使用 `Windows`平台上自己的语义解析来正常执行了:

    # escape=`

    FROM microsoft/nanoserver
    COPY testfile.txt c:\
    RUN dir c:\

Results in:

    PS C:\John> docker build -t succeeds --no-cache=true .
    Sending build context to Docker daemon 3.072 kB
    Step 1/3 : FROM microsoft/nanoserver
     ---> 22738ff49c6d
    Step 2/3 : COPY testfile.txt c:\
     ---> 96655de338de
    Removing intermediate container 4db9acbb1682
    Step 3/3 : RUN dir c:\
     ---> Running in a2c157f842f5
     Volume in drive C has no label.
     Volume Serial Number is 7E6D-E0F7

     Directory of c:\

    10/05/2016  05:04 PM             1,894 License.txt
    10/05/2016  02:22 PM    <DIR>          Program Files
    10/05/2016  02:14 PM    <DIR>          Program Files (x86)
    10/28/2016  11:18 AM                62 testfile.txt
    10/28/2016  11:20 AM    <DIR>          Users
    10/28/2016  11:20 AM    <DIR>          Windows
               2 File(s)          1,956 bytes
               4 Dir(s)  21,259,096,064 bytes free
     ---> 01c7f3bef04f
    Removing intermediate container a2c157f842f5
    Successfully built 01c7f3bef04f
    PS C:\John>

## Environment replacement

环境变量 (使用 `ENV` 语句进行声明) 可以被当做变量用到一些指令中，从而被 `Dockerfile`进行解析.

在 `Dockerfile`中使用`$variable_name` 或者 `${variable_name}` 来标记环境变量. 他们会被等效地处理，并且带有大括号的语法通常被用在没有空格的变量定义中，例如 `${foo}_bar`.

`${variable_name}` 语法还支持下面指定的一些标准`bash`修饰符:

* `${variable:-word}` 表示如果变量 `variable` 被设置了，那结果就是变量值. 如果变量 `variable` 没有被设置，那 `word` 就是变量值.
* `${variable:+word}` 表示如果变量 `variable`被设置了，那么 `word`就是变量值, 否则就是空值.

在所有情况下，`word` 可以是任何字符串，包括其他环境变量。

可以在变量之前添加 `\` 来进行转义:例如 `\$foo` 和 `\${foo}`将被转换成 `$foo` 和 `${foo}` 文本.

Example (`#`后面表示转义后的内容):

    FROM busybox
    ENV foo /bar
    WORKDIR ${foo}   # WORKDIR /bar
    ADD . $foo       # ADD . /bar
    COPY \$foo /quux # COPY $foo /quux

`Dockerfile` 中的以下指令列表支持环境变量:

* `ADD`
* `COPY`
* `ENV`
* `EXPOSE`
* `FROM`
* `LABEL`
* `STOPSIGNAL`
* `USER`
* `VOLUME`
* `WORKDIR`

以及:

* `ONBUILD` (与上面的任意一条指令组合使用的时候)

> **注意**:
> 1.4 之前的版本, `ONBUILD` 指令 **不** 不支持环境变量, 即使是与上面的指令混合使用的时候.

环境变量替换将在整个指令中对每个变量使用相同的值. 换句话说,在下面的例子中:

    ENV abc=hello
    ENV abc=bye def=$abc
    ENV ghi=$abc

 `def` 的值将会是 `hello`, 而不是 `bye`. 然而`ghi` 的值将会是 `bye` 。 因为它不是将 `abc` 设置为 `bye`的指令的一部分.可以理解为，环境变量是在下一个指令中生效。

## .dockerignore file

在 docker CLI 将上下文环境提交给 docker daemon的时候 , 它会在上下文根环境中找一个名为 `.dockerignore` 的文件.
如果这个文件存在,  CLI 会修改上下文排除`.dockerignore`文件中匹配的文件或者目录。 这有助于避免将不必要的大文件或者敏感性的文件传递给Docker deamon,并且可能使用 `ADD` or `COPY` 将这些文件添加到新的镜像中。

 `.dockerignore` 中定义的文件使用Unix Shell 的文件通配符匹配 .  出于匹配的目的，上下文的根被认为是工作目录和根目录.  例如, pattern`/foo/bar` 和 `foo/bar` 在 `PATH` 为 `foo`的子目录中或者位于`URL` 的 git repository  中排除掉 `bar`
 目录.

如果 `.dockerignore` 文件的某行以 `#` 开头, 这行会被认为是注释，并且不会被CLI解释执行.

Here is an example `.dockerignore` file:

```
# comment
*/temp*
*/*/temp*
temp?
```

This file causes the following build behavior:

| Rule        | Behavior                                                                                                                                                                                                       |
|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `# comment` | 忽略.  |
| `*/temp*`   | 在根环境的直接子目录中排除掉以 `temp`开头的文件和目录 .  例如, `/somedir/temporary.txt` 被排除, 目录 `/somedir/temp`也是如此. |
| `*/*/temp*` | 从根目录下两级的任何子目录中排除以`temp` 开头的文件和目录,例如, `/somedir/subdir/temporary.txt` 被排除. |
| `temp?`     | `?` 表示一个字符的替代，所以根目录下以`temp`开头的5个字符的目录或者文件会被排除掉. 例如, `/tempa` 和 `/tempb` 被排除掉.                                                     |


匹配是使用 Go语言的 [filepath.Match](http://golang.org/pkg/path/filepath#Match) 规则.  预处理过程是使用[filepath.Clean](http://golang.org/pkg/path/filepath/#Clean)清除掉 `.` and `..` 以及一些尾随的空格. 预处理后为空的行将被忽略.

除了 Go语言的 filepath.Match 规则之外,Docker还支持一个特殊的通配符字符串`**`，它匹配任意数量的目录（包括零）. 例如, `**/*.go` 将会排除在所有目录中找到的以`.go`结尾的所有文件，包括构建上下文的根.

以 `!` (感叹号) 可用于排除例外情况.  下面例子中 `.dockerignore`文件就是使用的这种机制：

```
    *.md
    !README.md
```

 *除了* `README.md` 之外的所有的markdown 文件都被排除在上下文之外.

 `!` 规则的位置可能会导致不同的影响:  `.dockerignore` 的最后一行匹配了规则， 决定了它会被包含还是排除.  考虑下下面的例子:

```
    *.md
    !README*.md
    README-secret.md
```

除了`README-secret.md`文件，其余所有的 README 文件都会被包含到上下文中，而其他的markdown文件则被排除掉.

再看下面的例子:

```
    *.md
    README-secret.md
    !README*.md
```

所有的 README 文件都会被包含在内. 中间那一行不会起到任何作用 ，`!README*.md` 匹配了 `README-secret.md`.

你甚至可以使用 `.dockerignore` 文件来排除 `Dockerfile`和 `.dockerignore` 文件. 这些文件仍然会被送给docker deamon 因为它需要他们来完成它的工作。  但是 `ADD` 和 `COPY` 不会把它们拷贝到镜像中.

最后，你可能希望指定要包含在上下文中的文件，而不是要排除的文件。可以将第一个模式指定为 `*` ,剩下的模式指定为 `!` .

**Note**: 由于历史原因,  `.` 模式被忽略了.

## FROM

    FROM <image> [AS <name>]

或者

    FROM <image>[:<tag>] [AS <name>]

或者

    FROM <image>[@<digest>] [AS <name>]

 `FROM` 指令初始化新的构建阶段并为后续指令设置基本*Base Image*. 因此有效的`Dockerfile`文件必须以 `FROM` 指令开头. 镜像可以是任何镜像 – 从  [*公共 Repositories*](https://docs.docker.com/engine/tutorials/dockerrepos/) 中**pulling an image**很容易.

- `ARG` 指令是`Dockerfile`中唯一能够在`FROM`之前的指令。

- `FROM` 指令可以在一个`Dockerfile` 中出现多次来创建多个镜像，或者 使用一个构建阶段作为另一个构建阶段的依赖项.
 只需要在  `FROM` 之前记下提交输出的最后一个image id . 每一个 `FROM` 会清除掉先前指令创建的任何状态.

- （可选购） 通过给  `FROM` 指令添加`AS name` 选项，可以给每个构建阶段指定一个名字. 该名称可以在后续的 `FROM` 指令和
  `COPY --from=<name|index>` 指令中使用，以引用此阶段构建的镜像。

-  `tag` 或者 `digest` 值是可选的. 如果找不到他们的值,默认是`latest` . 如果编译器找不到 `tag` 值，会返回一个错误。

### Understand how ARG and FROM interact

`FROM` 指令支持在第一个`FROM`指令之前由  `ARG` 指令声明的变量。.

```Dockerfile
ARG  CODE_VERSION=latest
FROM base:${CODE_VERSION}
CMD  /code/run-app

FROM extras:${CODE_VERSION}
CMD  /code/run-extras
```

`FROM`指令之前由 `ARG`指令声明的变量是在构建过程之外的，也就是说，这些变量不能被 `FROM` 指令之后的任何其他指令引用. 如果要使用 第一个`FROM` 指令之前由 `ARG` 指令声明的变量的值， 在指令之前再使用`ARG`指令声明一下这个变量，但是不要给其赋值就可以了。 :

```Dockerfile
ARG VERSION=latest
FROM busybox:$VERSION
ARG VERSION
RUN echo $VERSION > image_version
```

## RUN

RUN 指令有两种形式:

- `RUN <command>` (*shell* 格式 ,  command 在 shell 中运行, 在Linux中默认是 `/bin/sh -c` ，在windows中默认是 `cmd /S /C`)
- `RUN ["executable", "param1", "param2"]` (*exec* 格式)

 `RUN` 指令会在一个新的层上执行指令，并commit 结果. 已经commit了结果的镜像将会 在 `Dockerfile` 中定义好的下一个构建过程(next step)中被用到。.

使用 `RUN` 指令来分层构建，并提交结果到镜像中， 符合 Docker 的核心理念。 commit很容易，并且从一个image 构建历史中的任何一个点都能够容易的创建出 container , 就像源码控制一样.

 *exec* 格式可以有效避免shell 字符重叠(avoid shell string munging), 而且可以 在一个没有特定shell的镜像中 `RUN` 指令.

可以使用 `SHELL` 命令更改 shell 格式 的默认 *shell* 。

在 *shell* 格式中可以使用 `\` (backslash) 将一个 `RUN` 指令分开两行写。例如：

```
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'
```
他们与下面这一行是等价的:

```
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```

> **注意**:
> 要使用除了 '/bin/sh' 以外的其他shell ,要使用 *exec* 格式的命令传递需要指定的shell，例如,
> `RUN ["/bin/bash", "-c", "echo hello"]`

> **注意**:
> *exec* 命令格式被解析为JSON数组，这意味着你必须在单词周围使用双引号(")而不是单引号(')。
> 

> **注意**:
> 与 *shell* 格式不同,  *exec* 不会调用shell命令.这意味那些shell命令执行过程不会发生,例如
> `RUN [ "echo", "$HOME" ]` 不会对变量 `$HOME` 进行变量替换.如果想要实现这个过程的话，使用 *shell* 格式
> 或者直接运行shell命令, 例如: `RUN [ "sh", "-c", "echo $HOME" ]`.
> 当使用 ` exec ` 格式直接执行shell 命令时, 与 shell 格式一样，是shell 对命令中的变量进行了替换，而不是docker。

> **注意**:
> 在*JSON* 格式中,有必要转义反斜杠. 尤其是在windows中，反斜杠作为路径分隔符存在.
> 否则，下面的这行将会被当作*shell*格式来执行，而不是有效的*JSON*格式，从而会导致意想不到的错误:  
> `RUN ["c:\windows\system32\tasklist.exe"]`  
> 正确的写法是这样的:  
> `RUN ["c:\\windows\\system32\\tasklist.exe"]`

 `RUN` 指令缓存在下一个构建过程中不会自动失效. 例如像
`RUN apt-get dist-upgrade -y`指令的缓存在下一次构建过程中就会被重用. 使用 `--no-cache` 参数可以使  `RUN` 指令缓存失效，例如`docker build --no-cache`.


`ADD` 指令可以使 `RUN` 指令缓存失效. 

## CMD

`CMD` 指令含有三种形式:

- `CMD ["executable","param1","param2"]` (*exec* 格式, 首选)
- `CMD ["param1","param2"]` (作为 *default parameters to ENTRYPOINT*)
- `CMD command param1 param2` (*shell* 格式)

`Dockerfile`中只能有一个 `CMD` 指令 . 如果列举了多个 `CMD` 指令，只有最后一个会生效.

** `CMD` 指令的首要目的就是为一个容器提供可执行的默认值.** 这些默认值可以包括可执行文件，也可以省略可执行文件，省略可执行文件的话，就必须指定 `ENTRYPOINT`指令.

> **注意**:
> 如果 `CMD` 指令用来给 `ENTRYPOINT`指令提供默认参数的话,  `CMD` 和 `ENTRYPOINT` 指令都应该使用 JSON 数组的格式。

> **注意**:
> *exec* 格式指令会解析成JSON数组的格式, 这也就意味着每个词必须使用双引号 ` (") ` 而不是单引号 ` (') `.

> **注意**:
> 与*shell* 格式不同,  *exec* 格式不需要调用 shell 命令.
> 这也就意味着不会有shell命令执行过程发生. 例如,
> `CMD [ "echo", "$HOME" ]`，不会对变量 `$HOME` 进行变量替换.
> 如果想要用shell来处理的话，可以使用 *shell* 格式，或者直接执行一个shell命令
> , 例如: `CMD [ "sh", "-c", "echo $HOME" ]`.
> 当使用shell格式的命令或者直接执行了shell命令的时候，是shell对命令中的变量进行了替换，不是docker.

当使用了 shell 或者 exec 格式时,  `CMD` 指令制定的命令会在运行镜像的时候被执行(executed when running the image).

如果使用了 `CMD` 命令的 *shell* 格式 ,  `<command>` 将会在`/bin/sh -c` 中执行:

    FROM ubuntu
    CMD echo "This is a test." | wc -

如果想要 **不在shell中运行** `<command>`, 就需要使用  JSON array 的形式，同时指定可执行程序的全路径.
**这种JSON Array的形式是 `CMD` 命令的首选格式.** 任何附加参数都必须单独表示为数组中的字符串:

    FROM ubuntu
    CMD ["/usr/bin/wc","--help"]

如果希望容器每次都运行相同的可执行文件,那可以考虑将 `ENTRYPOINT` 和 `CMD` 结合起来使用. 

如果用户为 `docker run` 命令指定了参数，那么他们覆盖掉 `CMD`命令的默认参数.

> **注意**:
> 不要混淆了 `RUN` 和 `CMD`. `RUN` 实际上是运行了一个指令并commit 了结果; `CMD` 在构建的时候，不执行任何的指令，而是指定了镜像在运行时的一个预期的命令.

## LABEL

    LABEL <key>=<value> <key>=<value> <key>=<value> ...

 `LABEL` 指令为镜像添加了一些元数据. 一个 `LABEL` 就是一个键值对. 如果要在 `LABEL` 中包含空格,请像在命令行解析中那样使用引号和反斜杠 . 例如下面的使用示例:

    LABEL "com.example.vendor"="ACME Incorporated"
    LABEL com.example.label-with-value="foo"
    LABEL version="1.0"
    LABEL description="This text illustrates \
    that label-values can span multiple lines."

一个镜像可以有多个label标签. 也可以在一行当中指定多个label . 在Docker 1.10之前 这样能够减少镜像的大小，但是现在不会了. 下面就是这两种示例:

```none
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

```none
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

在base镜像和父镜像(images in the `FROM` line) 也会被我们的镜像继承过来. 如果一个镜像已经存在了但是有不一样的值,最新指定的值会覆盖之前指定的值.

可以使用`docker inspect` 命令来查看docker镜像中的值.

    "Labels": {
        "com.example.vendor": "ACME Incorporated"
        "com.example.label-with-value": "foo",
        "version": "1.0",
        "description": "This text illustrates that label-values can span multiple lines.",
        "multi.label1": "value1",
        "multi.label2": "value2",
        "other": "value3"
    },

## MAINTAINER (过时了)

    MAINTAINER <name>

 `MAINTAINER` 指令可以设置构建这个镜像的  *Author* . `LABEL` 指令是一个更灵活的版本，现在应该使用它来代替 `MAINTAINER`, 它能够设置我们需要的任何元数据, 同时使用 `docker inspect`也很容易被查看,例如使用 `LABEL` 来设置 
`MAINTAINER` 字段的话，可以像下面这样:

    LABEL maintainer="SvenDowideit@home.org.au"

这样的话，使用`docker inspect`命令就能够看到这条 label 标签.

## EXPOSE

    EXPOSE <port> [<port>/<protocol>...]

`EXPOSE` 指令通知Docker容器在运行时监听指定的网络端口. 我们可以指定监听TCP 或者 UDP 端口, 如果不指定的话，默认是TCP协议.

 `EXPOSE` 指令实际上不发布端口. 它作为一种文档类型在构建映像的人员和运行容器的人员之间运行，这些人打算发布哪些端口 . 想要在运行容器时实际发布这些端口，`docker run` 命令加上 `-p` 来发布和映射一个或者多个端口，或者使用 `-P` 来发布所有指定的端口或者把他们映射到更高级的端口上。

为了将端口重定向到宿主机系统上可以查看 ` -P ` 参数的详细用法.  `docker network` 命令支持创建一个用于容器间交流的网络，而不用开放指定的端口，因为容器可以加入到这个网络并通过任意的端口进行通信。可以点击下面地址查看详细内容,
[overview of this feature](https://docs.docker.com/engine/userguide/networking/).

## ENV

    ENV <key> <value>
    ENV <key>=<value> ...

 `ENV` 指令用来设置环境变量，将 `<key>` 指定值为
`<value>`.  此值将位于`Dockerfile`所有“后代”命令的环境中，并且可以在许多内联中进行内联替换.

 `ENV`指令有两种格式. 第一种, `ENV <key> <value>`,
设置一个单独的变量值. 空格之后的整个字符串都会被当作 `<value>` 来对待 - 包括宝行的控制或者引号.

第二种格式, `ENV <key>=<value> ...`, 允许一次设置多个变量值.注意，这种形式需要使用在语法中使用 (=) , 第一种形式不需要. 与命令行解析一样，引号和反斜杠可用于在值内包含空格.

例如:

    ENV myName="John Doe" myDog=Rex\ The\ Dog \
        myCat=fluffy

and

    ENV myName John Doe
    ENV myDog Rex The Dog
    ENV myCat fluffy

会在最终的镜像中产生相同的结果, 但是一种为首选，因为会产生一个单独的缓存层.

当从生成的图像运行容器时，使用`ENV`设置的环境变量将保持不变. 您可以使用`docker inspect`查看值，并使用`docker run --env <key> = <value>更改它们.

> **注意**:
> 环境持久性可能会导致意外的副作用. 例如,
> 设置了 `ENV DEBIAN_FRONTEND noninteractive` 可能会在基于Debian的图像上混淆apt-get用户. 要为单个命令设置值，请使用`RUN <key> = <value> <command>`.

## ADD

ADD有两种形式:

- `ADD [--chown=<user>:<group>] <src>... <dest>`
- `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]` (如果路径中含有空格需要使用这种形式)

> **注意**:
>  `--chown` 特征仅在用于构建Linux容器的Dockerfiles上受支持,windows 容器上将会不起作用.由于用户和组所有权概念不能在Linux和Windows之间进行转换，因此使用`/etc/passwd`和`/etc/group`将用户名和组名转换为ID会限制此功能仅适用于基于Linux的容器.

The `ADD` 指令，拷贝新文件、目录或者 `<src>`指定的远程文件，并将他们添加到 `<dest>` 指定的镜像的文件系统中.

可以指定多个`<src>`资源，但如果它们是文件或目录，它们的路径被解释基于构建的上下文的相对路径。  


每个 `<src>` 可以包含通配符，匹配工作将使用 go  语言的
[filepath.Match](http://golang.org/pkg/path/filepath#Match) 规则来完成. 例如:

    ADD hom* /mydir/        # adds all files starting with "hom"
    ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

`<dest>`是一个绝对路径，或者是相对于`WORKDIR`的路径,源文件将会被复制到目标容器中.

    ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
    ADD test /absoluteDir/         # adds "test" to /absoluteDir/

当添加含有特殊字符(例如`[`和 `]`)的文件或者目录时 , 需要遵循Golang规则来逃避这些路径，以防止它们被视为匹配模式. 例如, 添加一个名为 `arr[0].txt` 的文件时,使用下面的方式;

    ADD arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/


创建出来的新文件和目录的 UID 和 GID 都是 0, 除非使用 `--chown` 参数来指定 username, groupname, 或者使用 UID/GID 组合来指定添加内容的属主和属组.`--chown` 格式允许 使用用户名和组名字符串或者直接使用UID 和 GID 的数字组合.如果只提供用户名，不提供组名,默认将用户名作为组名，同理，如果只提供UID，不提供GID的话，默认将UID作为GID. 如果提供了用户名或者组名, 容器的根文件系统会根据`/etc/passwd` 和 `/etc/group` 文件将名称分别转换成 UID 或者 GID. 下面示例演示了如何使用 `--chown` 参数:

    ADD --chown=55:mygroup files* /somedir/
    ADD --chown=bin files* /somedir/
    ADD --chown=1 files* /somedir/
    ADD --chown=10:11 files* /somedir/

如果容器的根文件系统不包含 `/etc/passwd` 或者`/etc/group` 文件，并且也没有在 `--chown` 参数中指定属主属组, 将会在`ADD`操作这里构建失败. 使用数字ID不需要查找，也不依赖于容器根文件系统内容.

在 `<src>` 是远程文件目录的情况下, 目标位置需要具备600权限. 如果正在检索的远程文件具有HTTP `Last-Modified`标头，则该标头的时间戳将用于在目标文件上设置`mtime`. 但是，像在 `ADD`期间处理的其他文件一样, `mtime` 不会被作为确定文件是否已更改并且应更新缓存的依据 .

> **注意**:
> 如果通过传递`Dockerfile`到STDIN(`docker build - <somefile`)来构建镜像，则没有构建上下文, 所以 `Dockerfile`只能包含基于URL的`ADD`指令. 也可以通过 STDIN 传送一个位于 `Dockerfile` 根目录下的压缩文件: (`docker build - < archive.tar.gz`),压缩文档的其余部分将会作为这次构建的上下文.

> **注意**:
> 如果URL文件使用身份验证进行保护，则需要使用`RUN wget`，`RUN curl`或使用容器内的其他工具，因为`ADD`指令不支持身份验证.

> **注意**:
> 如果`<src>`的内容已经改变，第一次遇到`ADD`指令将使Dockerfile的所有后续指令的高速缓存无效。这包括使`RUN`指令的高速缓存无效。




`ADD` 遵守下列规则:

-  `<src>` 路径必须位于 *构建上下文* 中;不能像 `ADD ../something /something` 这样来指定, 因为  `docker build`的第一步就是将 上下文目录 (包含子目录) 发送给
  docker daemon.

- 如果`<src>`是一个URL并且`<dest>`没有以斜杠结束，则从URL下载文件并复制到`<dest>`.

- 如果`<src>`是一个URL并且`<dest>`以尾部斜杠结尾，则从URL推断出文件名，并将文件下载到`<dest>/<filename>`. 例如, `ADD http://example.com/foobar /` 将创建文件`/foobar`.  URL 必须有一个普通路径，以便能够找到一个合适的文件名字,这种情况(`http://example.com`)将不会起作用.

- 如果 `<src>` 是一个目录, 目录的整体内容都会被拷贝，包括文件系统元数据.

> **注意**:
> 文件目录本身不会被拷贝, 仅仅拷贝它的内容.

- 如果 `<src>` 是一个 *本地的* 使用公认的压缩格式(identity, gzip, bzip2 or xz) 的tar包，它会被解压缩成一个目录. *远程* URLs的资源 **不** 解压缩. 当一个目录被拷贝或者解压缩时, 具有与`tar -x`命令一致的效果 ,包含下面几种:

    1. Whatever existed at the destination path and
    2. The contents of the source tree, with conflicts resolved in favor
       of "2." on a file-by-file basis.

  > **注意**:
  > 是否将文件标识为已识别的压缩格式仅基于文件的内容而不是文件的名称来完成.
  > 例如，如果空文件恰好以`.tar.gz`结尾，则不会将其识别为压缩文件，**不会**生成任何类型的解压缩错误消息，而是将文件简单地复制到 目的地.

- 如果`<src>`是任何其他类型的文件，它将与其元数据一起单独复制。 在这种情况下，如果`<dest>`以尾部斜杠`/`结尾，它将被视为一个目录，`<src>`的内容将写在`<dest>/base(<src>)`.

- 如果直接或者使用通配符制定了多个 `<src>`资源,  `<dest>` 就必须是一个路径, 而且必须以 `/`结尾.

- 如果 `<dest>` 不以斜杠结尾, 它将被视为一个常规文件，`<src>` 的内容将会写在 `<dest>` 上.

- 如果 `<dest>` 不存在, 它将与路径中所有缺少的目录一起被创建.

## COPY

COPY 指令有两种形式:

- `COPY [--chown=<user>:<group>] <src>... <dest>`
- `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]` (如果路径中含有空格需要使用这种形式)

> **注意**:
>  `--chown` 特征仅在用于构建Linux容器的Dockerfiles上受支持,windows 容器上将会不起作用.由于用户和组所有权概念不能在Linux和Windows之间进行转换，因此使用`/etc/passwd`和`/etc/group`将用户名和组名转换为ID会限制此功能仅适用于基于Linux的容器.

`COPY` 指令 从 `<src>` 拷贝文件和目录，并把他们添加到容器的文件系统`<dest>`目标路径中。

可以指定多个 `<src>` 资源路径，但是文件和目录的路径视为构建上下文的相对路径.

每个 `<src>` 可以包含通配符，匹配工作将使用 go  语言的[filepath.Match](http://golang.org/pkg/path/filepath#Match) 规则来完成. 例如:

    COPY hom* /mydir/        # adds all files starting with "hom"
    COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

 `<dest>`是一个绝对路径，或者是相对于`WORKDIR`的路径,源文件将会被复制到目标容器中.

    COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
    COPY test /absoluteDir/  # adds "test" to /absoluteDir/


当添加含有特殊字符(例如`[`和 `]`)的文件或者目录时 , 需要遵循Golang规则来逃避这些路径，以防止它们被视为匹配模式. 例如, 添加一个名为 `arr[0].txt` 的文件时,使用下面的方式;;

    COPY arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/

创建出来的新文件和目录的 UID 和 GID 都是 0, 除非使用 `--chown` 参数来指定 username, groupname, 或者使用 UID/GID 组合来指定添加内容的属主和属组.`--chown` 格式允许 使用用户名和组名字符串或者直接使用UID 和 GID 的数字组合.如果只提供用户名，不提供组名,默认将用户名作为组名，同理，如果只提供UID，不提供GID的话，默认将UID作为GID. 如果提供了用户名或者组名, 容器的根文件系统会根据`/etc/passwd` 和 `/etc/group` 文件将名称分别转换成 UID 或者 GID. 下面示例演示了如何使用 `--chown` 参数:

    COPY --chown=55:mygroup files* /somedir/
    COPY --chown=bin files* /somedir/
    COPY --chown=1 files* /somedir/
    COPY --chown=10:11 files* /somedir/

如果容器的根文件系统不包含 `/etc/passwd` 或者`/etc/group` 文件，并且也没有在 `--chown` 参数中指定属主属组, 将会在`COPY`操作这里构建失败. 使用数字ID不需要查找，也不依赖于容器根文件系统内容.

> **注意**:
>  如果通过传递`Dockerfile`到STDIN(`docker build - <somefile`)来构建镜像，由于没有构建上下文，`COPY` 指令不能使用.

(可选) `COPY` 可以通过参数 `--from=<name|index>` 指定前一个构建阶段 (created with `FROM .. AS <name>`)的源路径，来作为构建上下文，而不使用用户默认的。这个参数还支持使用`FROM`指令启动的所有先前构建阶段分配的数字索引 . 如果找不到具有指定名称的构建阶段，则尝试使用具有相同名称的镜像.

`COPY` 遵守以下规则:

-  `<src>` 目录必须位于构建*上下文*; 不能够直接操作 `COPY ../something /something`, 因为`docker build`的第一步就是把上下文路径(包括子路径) 发送给 docker daemon.

-  如果 `<src>` 是一个目录, 这个目录包含的所有内容都会被复制,包括文件系统元数据.

> **注意**:
> 目录本身不会被拷贝, 只拷贝它的内容.

- 如果`<src>`是任何其他类型的文件，它将与其元数据一起单独复制。 在这种情况下，如果`<dest>`以尾部斜杠`/`结尾，它将被视为一个目录，`<src>`的内容将写在`<dest>/base(<src>)`.

- 如果直接或者使用通配符制定了多个 `<src>`资源,  `<dest>` 就必须是一个路径, 而且必须以 `/`结尾.

- 如果 `<dest>` 不以斜杠结尾, 它将被视为一个常规文件，`<src>` 的内容将会写在 `<dest>` 上.

- 如果 `<dest>` 不存在, 它将与路径中所有缺少的目录一起被创建.

## ENTRYPOINT

ENTRYPOINT 指令具有两种格式:

- `ENTRYPOINT ["executable", "param1", "param2"]`(*exec* 格式, 首选)
- `ENTRYPOINT command param1 param2`(*shell* 格式)

`ENTRYPOINT` 允许配置容器并将容器作为可执行文件来使用.

例如, 下面的指令可以启动一个nginx,运行着默认的内容，监听在80端口:

    docker run -i -t --rm -p 80:80 nginx

`docker run <image>` 的命令行参数 会被追加到`ENTRYPOINT` 指令的 *exec* 格式 的所有元素后面 ,并且会重写 `CMD` 指令指定的所有元素.
这允许参数被传递到入口处, 即, `docker run <image> -d`会将 `-d` 参数传递到入口处.可以使用`docker run --entrypoint`命令来重写 `ENTRYPOINT` 指令.

 *shell* 命令格式禁止使用任何 `CMD` 或者 `run` 命令行参数, 但缺点就是 `ENTRYPOINT` 会以 `/bin/sh -c` 这样的一个子命令启动, 这个命令不能传递任何参数.
这意味可执行程序将不会是容器的 `PID 1` - 并且 将不会接收 Unix 信号 - 所以我们的可执行程序将不能从`docker stop <container>`指令中接收到 `SIGTERM` 指令 .

只有 `Dockerfile`中最后一个 `ENTRYPOINT` 指令会起作用.

### Exec form ENTRYPOINT example

我们可以使用 `ENTRYPOINT`指令的 *exec* 格式来设定一些非常稳定的默认的指令或者参数，然后使用 `CMD` 来设置附加的很有可能被改掉的默认值.

    FROM ubuntu
    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]

当运行容器时, 能够看到 `top` 是唯一的进程:

    $ docker run -it --rm --name test  top -H
    top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
    Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
    KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

      PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
        1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top

要进一步检查结果, 可以使用 `docker exec`:

    $ docker exec -it test ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
    root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux

并且我们使用`docker stop test` 优雅地要求 `top` 关闭.

下面的`Dockerfile` 示例演示了 如何使用 `ENTRYPOINT` 指令来在前台运行 Apache  (即, 作为 `PID 1`):

```
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

如果需要为一个可执行程序写一个启动脚本, 您可以使用`exec`和`gosu`命令确保最终可执行文件接收Unix信号:

```bash
#!/usr/bin/env bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

最后, 如果需要在shutdown时做一些清理工作(或者与其他容器进行沟通)
, 或者协调多个可执行文件, 我们需要确保 `ENTRYPOINT` 脚本能够接收到 Unix信号,并传递它们，然后执行更多的工作 :

```
#!/bin/sh
# Note: I've written this using sh so it works in the busybox container too

# USE the trap if you need to also do manual cleanup after the service is stopped,
#     or need to start multiple services in the one container
trap "echo TRAPed signal" HUP INT QUIT TERM

# start service in background here
/usr/sbin/apachectl start

echo "[hit enter key to exit] or run 'docker stop <container>'"
read

# stop service and clean up here
echo "stopping apache"
/usr/sbin/apachectl stop

echo "exited $0"
```

如果使用`docker run -it --rm -p 80:80 --name test apache`命令运行了一个镜像 ,可以使用 `docker exec`, or `docker top` 来检查容器进程,
然后告诉脚本来停止 Apache:

```bash
$ docker exec -it test ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.0   4448   692 ?        Ss+  00:42   0:00 /bin/sh /run.sh 123 cmd cmd2
root        19  0.0  0.2  71304  4440 ?        Ss   00:42   0:00 /usr/sbin/apache2 -k start
www-data    20  0.2  0.2 360468  6004 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
www-data    21  0.2  0.2 360468  6000 ?        Sl   00:42   0:00 /usr/sbin/apache2 -k start
root        81  0.0  0.1  15572  2140 ?        R+   00:44   0:00 ps aux
$ docker top test
PID                 USER                COMMAND
10035               root                {run.sh} /bin/sh /run.sh 123 cmd cmd2
10054               root                /usr/sbin/apache2 -k start
10055               33                  /usr/sbin/apache2 -k start
10056               33                  /usr/sbin/apache2 -k start
$ /usr/bin/time docker stop test
test
real	0m 0.27s
user	0m 0.03s
sys	0m 0.03s
```

> **注意:** 可以使用`--entrypoint`参数来重写 `ENTRYPOINT` 设置,
> 但是这个只能将二进制文件指定给 *exec* (不会使用 `sh -c` ).

> **注意**:
>  *exec* 格式最终会被解析成 JSON 数组, 也就意味只
> 需要使用双引号(") 将词括起来，而不是单引号 (').

> **注意**:
> 与 *shell* 格式不同,  *exec* 格式不调用命令shell.
> 这也就意味着正常的shell处理过程不会发生. 例如,
> `ENTRYPOINT [ "echo", "$HOME" ]` 不会对 `$HOME`进行变量替换.
> 如果想要shell来处理，要么使用 *shell* 格式要么直接执行shell
> , 例如: `ENTRYPOINT [ "sh", "-c", "echo $HOME" ]`.
> 当使用shell格式的命令或者直接执行了shell命令的时候，是shell对命令中的变量进行了替换，不是docker.

### Shell form ENTRYPOINT example

可以为`ENTRYPOINT` 指定一个普通字符串，它将在`/bin/sh -c`中执行.
这样的话，将使用shell来替换shell环境变量，并且忽略`CMD` 指令或者 `docker run`命令的命令行参数.
为了确保 `docker stop` 命令能够直接给任何长时间运行的 `ENTRYPOINT`可执行程序发送指令,需要使用`exec`命令格式启动它:

    FROM ubuntu
    ENTRYPOINT exec top -b

当你运行这个镜像时, 会看到一个单独的 `PID 1` 进程:

    $ docker run -it --rm --name test top
    Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
    CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
    Load average: 0.08 0.03 0.05 2/98 6
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     R     3164   0%   0% top -b

该进程会在`docker stop`时被清理掉:

    $ /usr/bin/time docker stop test
    test
    real	0m 0.20s
    user	0m 0.02s
    sys	0m 0.04s

如果忘记了在`ENTRYPOINT`指令开始之前加上 `exec`的话:

    FROM ubuntu
    ENTRYPOINT top -b
    CMD --ignored-param1

可以运行它 (指定一个名字，留在下一步使用):

    $ docker run -it --name test top --ignored-param2
    Mem: 1704184K used, 352484K free, 0K shrd, 0K buff, 140621524238337K cached
    CPU:   9% usr   2% sys   0% nic  88% idle   0% io   0% irq   0% sirq
    Load average: 0.01 0.02 0.05 2/101 7
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     S     3168   0%   0% /bin/sh -c top -b cmd cmd2
        7     1 root     R     3164   0%   0% top -b

我们可以从`top` 命令的输出中看到， `ENTRYPOINT`  指令指定的程序不是 `PID 1`.

如果你运行 `docker stop test`, 容器不会退出并清理干净 - 
`stop` 命令会在 timeout之后发送一个 command  `SIGKILL` 命令，强制退出。:

    $ docker exec -it test ps aux
    PID   USER     COMMAND
        1 root     /bin/sh -c top -b cmd cmd2
        7 root     top -b
        8 root     ps aux
    $ /usr/bin/time docker stop test
    test
    real	0m 10.19s
    user	0m 0.04s
    sys	0m 0.03s

### 了解 CMD 和 ENTRYPOINT 如何相互作用

 `CMD` 和 `ENTRYPOINT`指令都定义了运行容器时执行的命令。.
很少有规则描述他们的相互作用关系.

1. Dockerfile 应至少指定一个`CMD` or `ENTRYPOINT` 命令.

2. 使用容器作为可执行文件时，应定义`ENTRYPOINT`.

3. `CMD`应该用作定义`ENTRYPOINT`命令的默认参数或在容器中执行ad-hoc命令的方法.

4. 使用备用参数运行容器时，将覆盖`CMD`命令的参数.

下表显示了针对不同的`ENTRYPOINT`/`CMD`组合，执行了哪些命令:

|                                | No ENTRYPOINT              | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT ["exec_entry", "p1_entry"]          |
|:-------------------------------|:---------------------------|:-------------------------------|:-----------------------------------------------|
| **No CMD**                     | *error, not allowed*       | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry                            |
| **CMD ["exec_cmd", "p1_cmd"]** | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd            |
| **CMD ["p1_cmd", "p2_cmd"]**   | p1_cmd p2_cmd              | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd              |
| **CMD exec_cmd p1_cmd**        | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |

## VOLUME

    VOLUME ["/data"]

`VOLUME`指令创建具有指定名称的挂载点，并将其标记为本地主机或者其他容器的存储卷。
值可以是JSON 数组, `VOLUME ["/var/log/"]`, 也可以是带有多个参数的普通字符串, 
例如 `VOLUME /var/log` 或者 `VOLUME /var/log/var/db`. 
有关Docker客户端的更多信息/示例和安装说明, 可以参考
[*Share Directories via Volumes*](https://docs.docker.com/engine/tutorials/dockervolumes/#/mount-a-host-directory-as-a-data-volume)
文档.

`docker run`命令使用base image 中指定位置存在的任何数据初始化新创建的卷. 例如,
如下的 Dockerfile 片段:

    FROM ubuntu
    RUN mkdir /myvol
    RUN echo "hello world" > /myvol/greeting
    VOLUME /myvol

这个 Dockerfile 镜像的结果就是导致`docker run`在`/ myvol`创建一个新的挂载点 导致`docker run`在`/ myvol`创建一个新的挂载点.

### Notes about specifying volumes

关于`Dockerfile`中的卷，请记住以下几点.

- **Volumes on Windows-based containers**:使用基于Windows的容器时,
  容器内卷的目标必须是下面其中一个:

  - 一个不存在的或者空的目录
  - 除了`C:`之外的驱动

- **Changing the volume from within the Dockerfile**: 如果任何一个构建过程在卷声明之后，修改了卷里面的数据，这些改动将会被忽略掉.

- **JSON formatting**: 如果列表被解析成 JSON 数组.必须用双引号（`“`）而不是单引号（```）.

- **The host directory is declared at container run-time**: 主机目录
  (挂载点) , 本质上, 是依赖主机的. 这是为了保持镜像的可移植性，因为不能保证给定的主机目录在所有主机上都可用. 因此, 不能从Dockerfile中关在主机目录。`VOLUME`指令不支持指定`host-dir`参数. 您必须在创建或运行容器时指定挂载点.

## USER

    USER <user>[:<group>]
or
    USER <UID>[:<GID>]

 运行镜像时 ,`USER` 指令用来指定`RUN`, `CMD` 和`ENTRYPOINT` 指令运行需要的属主(或者UID)属组(可选)。`USER` 指令在 `Dockerfile`中 放置在`RUN`, `CMD` 和`ENTRYPOINT` 指令之前.

> **注意**:
> 如果user 没有一个属组的话，镜像(或者下一条指令) 将会以 `root` 组来运行.

> 在Windows上, 如果不是内建账户的话，就一定要先创建用户账户.
> 这可以通过作为Dockerfile的一部分调用的`net user`命令来完成.

```Dockerfile
    FROM microsoft/windowsservercore
    # Create Windows user in the container
    RUN net user /add patrick
    # Set it for subsequent commands
    USER patrick
```


## WORKDIR

    WORKDIR /path/to/workdir

 `WORKDIR` 指令 为`Dockerfile`中后面跟着的 `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` 指令设定工作目录.
如果 `WORKDIR` 不存在, 它将被创建，即使它没有在任何后续的`Dockerfile`指令中使用.

`WORKDIR` 指令可以在`Dockerfile`中被使用多次. 如果指定的时相对路径, 将会被解析成是针对上一个`WORKDIR` 指令的相对路径. 例如:

    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd

在这个 `Dockerfile` 中`pwd`命令最终的输出结果将会是`/a/b/c`.

`WORKDIR`指令可以解析先前使用`ENV`设置的环境变量. 但是只能使用`Dockerfile`中显式设置的环境变量.
例如:

    ENV DIRPATH /path
    WORKDIR $DIRPATH/$DIRNAME
    RUN pwd

在这个 `Dockerfile` 中`pwd`命令最终的输出结果将会是`/path/$DIRNAME`

## ARG

    ARG <name>[=<default value>]

`ARG`指令使用`--build-arg <varname>=<value>`参数定义一个变量，用户可以使用`docker build`命令在构建时将其传递给构建器。
如果用户指定了未在Dockerfile中定义的构建参数，则构建会输出警告。

```
[Warning] One or more build-args [foo] were not consumed.
```

一个Dockerfile可以包含多个 `ARG` 指令. 例如,下面的Dockerfile也是可以的:

```
FROM busybox
ARG user1
ARG buildno
...
```

> **警告:** 建议不要使用构建时变量来传递github密钥，用户凭证等秘密. 
>  使用`docker history`命令，任何镜像用户都可以看到构建时变量值.

### Default values

(可选)`ARG` 指令可以有一个默认值:

```
FROM busybox
ARG user1=someuser
ARG buildno=1
...
```

如果 `ARG` 指令有了默认值，并且在构建时没有传递别的值，构建器将使用默认值.

### Scope

`ARG`变量定义从`Dockerfile`中定义的行开始生效，而不是来自命令行或其他地方的参数的使用. 例如下面的 Dockerfile:

```
1 FROM busybox
2 USER ${user:-some_user}
3 ARG user
4 USER $user
...
```
用户可以调用下面的命令构建这个文件:

```
$ docker build --build-arg user=what_user .
```

第2行的`USER`评估为`some_user`，因为`user`变量在后续第3行定义. 第四行的 `USER`被解释为`what_user` 因为 `user` 
已经定义了， `what_user` 在命令行中进行了传递. 在通过`ARG`指令定义之前，对变量的任何使用都会导致空字符串.

`ARG` 指令如果定义在构建过程的最后，会超出作用范围. 如果想在多个构建阶段使用参数，需要在每个阶段都包含`ARG`指令.

```
FROM busybox
ARG SETTINGS
RUN ./run/setup $SETTINGS

FROM busybox
ARG SETTINGS
RUN ./run/other $SETTINGS
```

### Using ARG variables

我们可以使用`ARG`或`ENV`指令来指定`RUN`指令可用的变量.使用`ENV`指令定义的环境变量总是覆盖`ARG`指令中定义的同名变量. 
下面的Dockerfile 示例中介绍了  `ENV` 和 `ARG` 指令的用法.

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER v1.0.0
4 RUN echo $CONT_IMG_VER
```
假如使用下面的命令来构建镜像的话:

```
$ docker build --build-arg CONT_IMG_VER=v2.0.1 .
```

在这种情况下, `RUN`指令使用`v1.0.0`而不是用户传递的`ARG`设置：`v2.0.1`。 
这种行为类似于shell脚本，从定义的角度来看，其中本地范围的变量覆盖作为参数传递的变量或从环境继承的变量.

参考上面的例子，使用一个不同的 `ENV` 格式，就可以结合 `ARG` 和 `ENV` 指令创造出很多有用的使用方式:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER ${CONT_IMG_VER:-v1.0.0}
4 RUN echo $CONT_IMG_VER
```

与 `ARG` 指令不同, `ENV` 的值，始终保留在构建的镜像中. 如果 docker build 时不加 `--build-arg` 参数:

```
$ docker build .
```

在这个示例中, `CONT_IMG_VER`仍然保留在镜像中，但其值为`v1.0.0`，因为它是`ENV`指令在第3行中的默认设置.


### Predefined ARGs 

Docker有一组预定义的`ARG`变量，我们可以在Dockerfile中直接使用，而不用指定相应的`ARG`指令。


* `HTTP_PROXY`
* `http_proxy`
* `HTTPS_PROXY`
* `https_proxy`
* `FTP_PROXY`
* `ftp_proxy`
* `NO_PROXY`
* `no_proxy`

要使用这些参数的话，只要在命令行中传递他们就可以:

```
--build-arg <varname>=<value>
```

默认情况下, 这些预定义的变量在`docker history`中是看不到的. 
这样可以降低在`HTTP_PROXY`变量中意外泄漏敏感验证信息的风险.

例如， 使用这个命令`--build-arg HTTP_PROXY=http://user:pass@proxy.lon.example.com` 来构建下面的Dockerfile：

``` Dockerfile
FROM ubuntu
RUN echo "Hello World"
```

此例中,`HTTP_PROXY` 变量的值在`docker history`中是不可用的，并且不会被缓存。 
如果需要更改位置，或者代理服务器需要更换为,`http://user:pass@proxy.sfo.example.com`, 后续构建中不会导致缓存未命中.

如果想要覆盖这个值， 可以像下面这样在Dockerfile中添加一个 `ARG`指令:

``` Dockerfile
FROM ubuntu
ARG HTTP_PROXY
RUN echo "Hello World"
```

构建这个Dockerfile时，`HTTP_PROXY`保存在`docker history`中，更改其值会使构建缓存无效.

### Impact on build caching

`ARG`变量不会像`ENV`变量一样持久存储在构建的图像中.
但是，`ARG`变量确实以类似的方式影响构建缓存. 
如果Dockerfile定义了一个'ARG`变量，其值与先前的构建不同，那么在第一次使用时会出现"cache miss"，而不是它的定义。
特别是，`ARG`指令后面的所有`RUN`指令都隐式使用`ARG`变量（作为环境变量），因此可能导致高速缓存未命中.
所有预定义的`ARG`变量都免于缓存，除非在`Dockerfile中有匹配的`ARG`语句.

例如下面两个 Dockerfile:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 RUN echo $CONT_IMG_VER
```

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 RUN echo hello
```

如果命令行中指定了 `--build-arg CONT_IMG_VER=<value>` 参数, 两种情况下, 第二行的定义不会导致 cache miss; 
第三行会导致 cache miss .`ARG CONT_IMG_VER` 导致 RUN 指令那一行被认为与 运行 `CONT_IMG_VER=<value>` 来 echo hello是一样的, 如果`<value>`
值发生变化了, 会有cache miss.

考虑同一命令的另一个示例:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER $CONT_IMG_VER
4 RUN echo $CONT_IMG_VER
```
在这个示例中, 第三行会发生 cache miss. 之所以缓存未命中是因为 because
`ENV`  中的变量值引用了`ARG` 变量，而这个变量在命令行中被改变了. 在这个示例中,  `ENV`
指令要求镜像包含这个值.

如果`ENV`指令覆盖了 `ARG` 指令中同名的变量, 例如 下面的Dockerfile:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER hello
4 RUN echo $CONT_IMG_VER
```

第3行不会导致缓存未命中，因为 `CONT_IMG_VER` 变量值是常量(`hello`). 因此，  `RUN`（第4行）上使用的环境变量和值在构建时不会发生变化.

## ONBUILD

    ONBUILD [INSTRUCTION]

当镜像用作另一个构建的基础镜像时,`ONBUILD`指令在镜像中添加一个* trigger *指令，以便稍后执行。
触发器将在下游构建的上下文中执行，就好像它是在下游`Dockerfile中的`FROM`指令之后立即插入的一样。

任何一个构建指令都可以作为触发器.

如果你正在构建的镜像还要作为另外一个镜像的基础镜像, 例如一个应用程序编译环境，或者用户自定义配置的守护进程。

例如, 如果你的镜像是一个可复用的 Python 应用程序构建器, 则需要将应用程序源代码添加到特定目录中, 
并在之后指定一个 build script. 您现在不能只调用`ADD`和`RUN`，因为您还无法访问应用程序源代码，并且每个应用程序构建都会有所不同. 
我们可以简单地为应用程序开发人员提供一个模板`Dockerfile`，来复制粘贴到他们的应用程序中，但这样做效率低，容易出错且难以更新，因为它与特定于应用程序的代码混合在一起.

解决方案是使用`ONBUILD`来准备一些需要提前的指令，以便在下一个构建阶段运行。T

下面是它的工作原理:

1. 当遇到`ONBUILD`指令时，构建器会为正在构建的镜像的元数据添加触发器。 该指令不会影响当前构建。
2. 在构建结束时,   镜像 清单的`OnBuild`键下面会存储一系列的触发器 . 他们可以通过 `docker inspect` 命令查看到.
3. 稍后，可以使用`FROM`指令将镜像用作新构建的基础. 
   作为 `FROM` 指令处理过程的一部分,下游构建器会查询 `ONBUILD` 触发器,然后按照他们注册的顺序执行他们。
   如果任何一个触发器构建失败的话,  `FROM` 指令会被终止这反过来导致构建失败. 
   如果所有的触发器都构建成功了, `FROM`指令完成，构建继续照常进行.
4. 执行后，触发器将从最终镜像中清除掉. 换句话说，他们不会被后续的构建过程所继承.

例如你可能会添加一些类似下面的内容:

    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]

> **注意**:  不允许使用 `ONBUILD ONBUILD`链接 `ONBUILD`指令.

> **注意**:   `ONBUILD` 指令可能不会触发`FROM` 或者 `MAINTAINER` 指令.

## STOPSIGNAL

    STOPSIGNAL signal

`STOPSIGNAL`指令设置将发送到容器，并使容器退出的系统调用信号.
此信号可以是与内核的系统调用表中的位置匹配的有效无符号数，例如9，或SIGNAME格式的信号名，例如SIGKILL.

## HEALTHCHECK

`HEALTHCHECK`指令有两种形式:

* `HEALTHCHECK [OPTIONS] CMD command` (通过在容器内运行命令来检查容器运行状况)
* `HEALTHCHECK NONE` (禁用从基础镜像继承的任何运行状况检查)

`HEALTHCHECK`指令告诉Docker如何测试容器以检查它是否仍在工作。
即使服务进程仍在运行，这也可以检测到陷入无限循环且无法处理新连接的Web服务器等情况.

当容器指定了运行状况检查时, 除了正常状态外，它还具有_health status_. 
这种状态最初是 `starting`。 每当健康检查通过时，它就变得`healthy`（无论以前处于什么状态）。
经过一定数量的连续失败后, 它将会变成 `unhealthy`.

在`CMD`之前可以出现的选项是:

* `--interval=DURATION` (default: `30s`)
* `--timeout=DURATION` (default: `30s`)
* `--start-period=DURATION` (default: `0s`)
* `--retries=N` (default: `3`)

健康检查将在容器启动后，隔几秒才开始运行，然后在上一次健康性检查完成之后，隔几秒再运行一次,以此重复.

如果检查的单次运行时间超时了，则认为检查失败.

对容器的健康性检查，重试几次都是失败的话， 容器的状态会被认为是 `unhealthy`.

**start period** provides initialization time for containers that need time to bootstrap.
Probe failure during that period will not be counted towards the maximum number of retries.
However, if a health check succeeds during the start period, the container is considered
started and all consecutive failures will be counted towards the maximum number of retries.

There can only be one `HEALTHCHECK` instruction in a Dockerfile. If you list
more than one then only the last `HEALTHCHECK` will take effect.

The command after the `CMD` keyword can be either a shell command (e.g. `HEALTHCHECK
CMD /bin/check-running`) or an _exec_ array (as with other Dockerfile commands;
see e.g. `ENTRYPOINT` for details).

The command's exit status indicates the health status of the container.
The possible values are:

- 0: success - the container is healthy and ready for use
- 1: unhealthy - the container is not working correctly
- 2: reserved - do not use this exit code

For example, to check every five minutes or so that a web-server is able to
serve the site's main page within three seconds:

    HEALTHCHECK --interval=5m --timeout=3s \
      CMD curl -f http://localhost/ || exit 1

To help debug failing probes, any output text (UTF-8 encoded) that the command writes
on stdout or stderr will be stored in the health status and can be queried with
`docker inspect`. Such output should be kept short (only the first 4096 bytes
are stored currently).

When the health status of a container changes, a `health_status` event is
generated with the new status.

The `HEALTHCHECK` feature was added in Docker 1.12.


## SHELL

    SHELL ["executable", "parameters"]

The `SHELL` instruction allows the default shell used for the *shell* form of
commands to be overridden. The default shell on Linux is `["/bin/sh", "-c"]`, and on
Windows is `["cmd", "/S", "/C"]`. The `SHELL` instruction *must* be written in JSON
form in a Dockerfile.

The `SHELL` instruction is particularly useful on Windows where there are
two commonly used and quite different native shells: `cmd` and `powershell`, as
well as alternate shells available including `sh`.

The `SHELL` instruction can appear multiple times. Each `SHELL` instruction overrides
all previous `SHELL` instructions, and affects all subsequent instructions. For example:

    FROM microsoft/windowsservercore

    # Executed as cmd /S /C echo default
    RUN echo default

    # Executed as cmd /S /C powershell -command Write-Host default
    RUN powershell -command Write-Host default

    # Executed as powershell -command Write-Host hello
    SHELL ["powershell", "-command"]
    RUN Write-Host hello

    # Executed as cmd /S /C echo hello
    SHELL ["cmd", "/S"", "/C"]
    RUN echo hello

The following instructions can be affected by the `SHELL` instruction when the
*shell* form of them is used in a Dockerfile: `RUN`, `CMD` and `ENTRYPOINT`.

The following example is a common pattern found on Windows which can be
streamlined by using the `SHELL` instruction:

    ...
    RUN powershell -command Execute-MyCmdlet -param1 "c:\foo.txt"
    ...

The command invoked by docker will be:

    cmd /S /C powershell -command Execute-MyCmdlet -param1 "c:\foo.txt"

This is inefficient for two reasons. First, there is an un-necessary cmd.exe command
processor (aka shell) being invoked. Second, each `RUN` instruction in the *shell*
form requires an extra `powershell -command` prefixing the command.

To make this more efficient, one of two mechanisms can be employed. One is to
use the JSON form of the RUN command such as:

    ...
    RUN ["powershell", "-command", "Execute-MyCmdlet", "-param1 \"c:\\foo.txt\""]
    ...

While the JSON form is unambiguous and does not use the un-necessary cmd.exe,
it does require more verbosity through double-quoting and escaping. The alternate
mechanism is to use the `SHELL` instruction and the *shell* form,
making a more natural syntax for Windows users, especially when combined with
the `escape` parser directive:

    # escape=`

    FROM microsoft/nanoserver
    SHELL ["powershell","-command"]
    RUN New-Item -ItemType Directory C:\Example
    ADD Execute-MyCmdlet.ps1 c:\example\
    RUN c:\example\Execute-MyCmdlet -sample 'hello world'

Resulting in:

    PS E:\docker\build\shell> docker build -t shell .
    Sending build context to Docker daemon 4.096 kB
    Step 1/5 : FROM microsoft/nanoserver
     ---> 22738ff49c6d
    Step 2/5 : SHELL powershell -command
     ---> Running in 6fcdb6855ae2
     ---> 6331462d4300
    Removing intermediate container 6fcdb6855ae2
    Step 3/5 : RUN New-Item -ItemType Directory C:\Example
     ---> Running in d0eef8386e97


        Directory: C:\


    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-----       10/28/2016  11:26 AM                Example


     ---> 3f2fbf1395d9
    Removing intermediate container d0eef8386e97
    Step 4/5 : ADD Execute-MyCmdlet.ps1 c:\example\
     ---> a955b2621c31
    Removing intermediate container b825593d39fc
    Step 5/5 : RUN c:\example\Execute-MyCmdlet 'hello world'
     ---> Running in be6d8e63fe75
    hello world
     ---> 8e559e9bf424
    Removing intermediate container be6d8e63fe75
    Successfully built 8e559e9bf424
    PS E:\docker\build\shell>

The `SHELL` instruction could also be used to modify the way in which
a shell operates. For example, using `SHELL cmd /S /C /V:ON|OFF` on Windows, delayed
environment variable expansion semantics could be modified.

The `SHELL` instruction can also be used on Linux should an alternate shell be
required such as `zsh`, `csh`, `tcsh` and others.

The `SHELL` feature was added in Docker 1.12.

## Dockerfile examples

Below you can see some examples of Dockerfile syntax. If you're interested in
something more realistic, take a look at the list of [Dockerization examples](https://docs.docker.com/engine/examples/).

```
# Nginx
#
# VERSION               0.0.1

FROM      ubuntu
LABEL Description="This image is used to start the foobar executable" Vendor="ACME Products" Version="1.0"
RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server
```

```
# Firefox over VNC
#
# VERSION               0.3

FROM ubuntu

# Install vnc, xvfb in order to create a 'fake' display and firefox
RUN apt-get update && apt-get install -y x11vnc xvfb firefox
RUN mkdir ~/.vnc
# Setup a password
RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
# Autostart firefox (might not be the best way, but it does the trick)
RUN bash -c 'echo "firefox" >> /.bashrc'

EXPOSE 5900
CMD    ["x11vnc", "-forever", "-usepw", "-create"]
```

```
# Multiple images example
#
# VERSION               0.1

FROM ubuntu
RUN echo foo > bar
# Will output something like ===> 907ad6c2736f

FROM ubuntu
RUN echo moo > oink
# Will output something like ===> 695d7793cbe4

# You'll now have two images, 907ad6c2736f with /bar, and 695d7793cbe4 with
# /oink.
```










































# Dockerfile reference

Docker 可以通过读`Dockerfile`中的指令自动构建镜像.`Dockerfile`是一个文本文档，其中包含用户可以在命令行上调用以组合图像的所有命令。. Using `docker build`
## FROM

- FROM 指令是最重要的一个且必须为Dockerfile文件开篇的第一个非注释行，用于为映像文件构建过程指定基准镜像，后续的指令运行于此基准镜像所提供的运行环境。
- 实践中，基准镜像可以是任何可用镜像文件，默认情况下，`docker build` 会在docker主机上查找指定的镜像文件，在其不存在的时候，则会从docker hub registry上拉取所需的镜像文件
    - 如果找不到指定的镜像文件，docker build 会返回一个错误信息。

- Syntax
    - FROM &lt;repository&gt;[:&lt;tag&gt;] 或
    - FROM &lt;repository&gt;@&lt;digest&gt;
        - &lt;repository&gt;: 指定的base image的名称;
        - &lt;tag&gt;：base image 的标签，为可选项，省略时默认是latest

## MAINTANIER(depreacted)

- 用于让Dockerfile制作者提供本人的详细信息
- dockerfile 并不限制MAINTANINER指令出现的位置，一般放置在FROM指令之后
- Synatx 
    - MAINTANIER &lt;author’s deatail&gt;
        - &lt;author’s deatail&gt; 可以是任何文本信息，但约定俗成地使用作者名称以及邮箱地址
        
