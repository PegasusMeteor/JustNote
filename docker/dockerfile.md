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

>
> **Note**:
> In the *JSON* form, it is necessary to escape backslashes. This is
> particularly relevant on Windows where the backslash is the path separator.
> The following line would otherwise be treated as *shell* form due to not
> being valid JSON, and fail in an unexpected way:
> `RUN ["c:\windows\system32\tasklist.exe"]`
> The correct syntax for this example is:
> `RUN ["c:\\windows\\system32\\tasklist.exe"]`

The cache for `RUN` instructions isn't invalidated automatically during
the next build. The cache for an instruction like
`RUN apt-get dist-upgrade -y` will be reused during the next build. The
cache for `RUN` instructions can be invalidated by using the `--no-cache`
flag, for example `docker build --no-cache`.

See the [`Dockerfile` Best Practices
guide](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#/build-cache) for more information.

The cache for `RUN` instructions can be invalidated by `ADD` instructions. See
[below](#add) for details.

### Known issues (RUN)

- [Issue 783](https://github.com/docker/docker/issues/783) is about file
  permissions problems that can occur when using the AUFS file system. You
  might notice it during an attempt to `rm` a file, for example.

  For systems that have recent aufs version (i.e., `dirperm1` mount option can
  be set), docker will attempt to fix the issue automatically by mounting
  the layers with `dirperm1` option. More details on `dirperm1` option can be
  found at [`aufs` man page](https://github.com/sfjro/aufs3-linux/tree/aufs3.18/Documentation/filesystems/aufs)

  If your system doesn't have support for `dirperm1`, the issue describes a workaround.

## CMD

The `CMD` instruction has three forms:

- `CMD ["executable","param1","param2"]` (*exec* form, this is the preferred form)
- `CMD ["param1","param2"]` (as *default parameters to ENTRYPOINT*)
- `CMD command param1 param2` (*shell* form)

There can only be one `CMD` instruction in a `Dockerfile`. If you list more than one `CMD`
then only the last `CMD` will take effect.

**The main purpose of a `CMD` is to provide defaults for an executing
container.** These defaults can include an executable, or they can omit
the executable, in which case you must specify an `ENTRYPOINT`
instruction as well.

> **Note**:
> If `CMD` is used to provide default arguments for the `ENTRYPOINT`
> instruction, both the `CMD` and `ENTRYPOINT` instructions should be specified
> with the JSON array format.

> **Note**:
> The *exec* form is parsed as a JSON array, which means that
> you must use double-quotes (") around words not single-quotes (').

> **Note**:
> Unlike the *shell* form, the *exec* form does not invoke a command shell.
> This means that normal shell processing does not happen. For example,
> `CMD [ "echo", "$HOME" ]` will not do variable substitution on `$HOME`.
> If you want shell processing then either use the *shell* form or execute
> a shell directly, for example: `CMD [ "sh", "-c", "echo $HOME" ]`.
> When using the exec form and executing a shell directly, as in the case for
> the shell form, it is the shell that is doing the environment variable
> expansion, not docker.

When used in the shell or exec formats, the `CMD` instruction sets the command
to be executed when running the image.

If you use the *shell* form of the `CMD`, then the `<command>` will execute in
`/bin/sh -c`:

    FROM ubuntu
    CMD echo "This is a test." | wc -

If you want to **run your** `<command>` **without a shell** then you must
express the command as a JSON array and give the full path to the executable.
**This array form is the preferred format of `CMD`.** Any additional parameters
must be individually expressed as strings in the array:

    FROM ubuntu
    CMD ["/usr/bin/wc","--help"]

If you would like your container to run the same executable every time, then
you should consider using `ENTRYPOINT` in combination with `CMD`. See
[*ENTRYPOINT*](#entrypoint).

If the user specifies arguments to `docker run` then they will override the
default specified in `CMD`.

> **Note**:
> Don't confuse `RUN` with `CMD`. `RUN` actually runs a command and commits
> the result; `CMD` does not execute anything at build time, but specifies
> the intended command for the image.

## LABEL

    LABEL <key>=<value> <key>=<value> <key>=<value> ...

The `LABEL` instruction adds metadata to an image. A `LABEL` is a
key-value pair. To include spaces within a `LABEL` value, use quotes and
backslashes as you would in command-line parsing. A few usage examples:

    LABEL "com.example.vendor"="ACME Incorporated"
    LABEL com.example.label-with-value="foo"
    LABEL version="1.0"
    LABEL description="This text illustrates \
    that label-values can span multiple lines."

An image can have more than one label. You can specify multiple labels on a
single line. Prior to Docker 1.10, this decreased the size of the final image,
but this is no longer the case. You may still choose to specify multiple labels
in a single instruction, in one of the following two ways:

```none
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

```none
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

Labels included in base or parent images (images in the `FROM` line) are
inherited by your image. If a label already exists but with a different value,
the most-recently-applied value overrides any previously-set value.

To view an image's labels, use the `docker inspect` command.

    "Labels": {
        "com.example.vendor": "ACME Incorporated"
        "com.example.label-with-value": "foo",
        "version": "1.0",
        "description": "This text illustrates that label-values can span multiple lines.",
        "multi.label1": "value1",
        "multi.label2": "value2",
        "other": "value3"
    },

## MAINTAINER (deprecated)

    MAINTAINER <name>

The `MAINTAINER` instruction sets the *Author* field of the generated images.
The `LABEL` instruction is a much more flexible version of this and you should use
it instead, as it enables setting any metadata you require, and can be viewed
easily, for example with `docker inspect`. To set a label corresponding to the
`MAINTAINER` field you could use:

    LABEL maintainer="SvenDowideit@home.org.au"

This will then be visible from `docker inspect` with the other labels.

## EXPOSE

    EXPOSE <port> [<port>/<protocol>...]

The `EXPOSE` instruction informs Docker that the container listens on the
specified network ports at runtime. You can specify whether the port listens on
TCP or UDP, and the default is TCP if the protocol is not specified.

The `EXPOSE` instruction does not actually publish the port. It functions as a
type of documentation between the person who builds the image and the person who
runs the container, about which ports are intended to be published. To actually
publish the port when running the container, use the `-p` flag on `docker run`
to publish and map one or more ports, or the `-P` flag to publish all exposed
ports and map them to high-order ports.

To set up port redirection on the host system, see [using the -P
flag](run.md#expose-incoming-ports). The `docker network` command supports
creating networks for communication among containers without the need to
expose or publish specific ports, because the containers connected to the
network can communicate with each other over any port. For detailed information,
see the
[overview of this feature](https://docs.docker.com/engine/userguide/networking/)).

## ENV

    ENV <key> <value>
    ENV <key>=<value> ...

The `ENV` instruction sets the environment variable `<key>` to the value
`<value>`. This value will be in the environment of all "descendant"
`Dockerfile` commands and can be [replaced inline](#environment-replacement) in
many as well.

The `ENV` instruction has two forms. The first form, `ENV <key> <value>`,
will set a single variable to a value. The entire string after the first
space will be treated as the `<value>` - including characters such as
spaces and quotes.

The second form, `ENV <key>=<value> ...`, allows for multiple variables to
be set at one time. Notice that the second form uses the equals sign (=)
in the syntax, while the first form does not. Like command line parsing,
quotes and backslashes can be used to include spaces within values.

For example:

    ENV myName="John Doe" myDog=Rex\ The\ Dog \
        myCat=fluffy

and

    ENV myName John Doe
    ENV myDog Rex The Dog
    ENV myCat fluffy

will yield the same net results in the final image, but the first form
is preferred because it produces a single cache layer.

The environment variables set using `ENV` will persist when a container is run
from the resulting image. You can view the values using `docker inspect`, and
change them using `docker run --env <key>=<value>`.

> **Note**:
> Environment persistence can cause unexpected side effects. For example,
> setting `ENV DEBIAN_FRONTEND noninteractive` may confuse apt-get
> users on a Debian-based image. To set a value for a single command, use
> `RUN <key>=<value> <command>`.

## ADD

ADD has two forms:

- `ADD [--chown=<user>:<group>] <src>... <dest>`
- `ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]` (this form is required for paths containing
whitespace)

> **Note**:
> The `--chown` feature is only supported on Dockerfiles used to build Linux containers,
> and will not work on Windows containers. Since user and group ownership concepts do
> not translate between Linux and Windows, the use of `/etc/passwd` and `/etc/group` for
> translating user and group names to IDs restricts this feature to only be viable for
> for Linux OS-based containers.

The `ADD` instruction copies new files, directories or remote file URLs from `<src>`
and adds them to the filesystem of the image at the path `<dest>`.

Multiple `<src>` resources may be specified but if they are files or
directories, their paths are interpreted as relative to the source of
the context of the build.

Each `<src>` may contain wildcards and matching will be done using Go's
[filepath.Match](http://golang.org/pkg/path/filepath#Match) rules. For example:

    ADD hom* /mydir/        # adds all files starting with "hom"
    ADD hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

The `<dest>` is an absolute path, or a path relative to `WORKDIR`, into which
the source will be copied inside the destination container.

    ADD test relativeDir/          # adds "test" to `WORKDIR`/relativeDir/
    ADD test /absoluteDir/         # adds "test" to /absoluteDir/

When adding files or directories that contain special characters (such as `[`
and `]`), you need to escape those paths following the Golang rules to prevent
them from being treated as a matching pattern. For example, to add a file
named `arr[0].txt`, use the following;

    ADD arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/


All new files and directories are created with a UID and GID of 0, unless the
optional `--chown` flag specifies a given username, groupname, or UID/GID
combination to request specific ownership of the content added. The
format of the `--chown` flag allows for either username and groupname strings
or direct integer UID and GID in any combination. Providing a username without
groupname or a UID without GID will use the same numeric UID as the GID. If a
username or groupname is provided, the container's root filesystem
`/etc/passwd` and `/etc/group` files will be used to perform the translation
from name to integer UID or GID respectively. The following examples show
valid definitions for the `--chown` flag:

    ADD --chown=55:mygroup files* /somedir/
    ADD --chown=bin files* /somedir/
    ADD --chown=1 files* /somedir/
    ADD --chown=10:11 files* /somedir/

If the container root filesystem does not contain either `/etc/passwd` or
`/etc/group` files and either user or group names are used in the `--chown`
flag, the build will fail on the `ADD` operation. Using numeric IDs requires
no lookup and will not depend on container root filesystem content.

In the case where `<src>` is a remote file URL, the destination will
have permissions of 600. If the remote file being retrieved has an HTTP
`Last-Modified` header, the timestamp from that header will be used
to set the `mtime` on the destination file. However, like any other file
processed during an `ADD`, `mtime` will not be included in the determination
of whether or not the file has changed and the cache should be updated.

> **Note**:
> If you build by passing a `Dockerfile` through STDIN (`docker
> build - < somefile`), there is no build context, so the `Dockerfile`
> can only contain a URL based `ADD` instruction. You can also pass a
> compressed archive through STDIN: (`docker build - < archive.tar.gz`),
> the `Dockerfile` at the root of the archive and the rest of the
> archive will be used as the context of the build.

> **Note**:
> If your URL files are protected using authentication, you
> will need to use `RUN wget`, `RUN curl` or use another tool from
> within the container as the `ADD` instruction does not support
> authentication.

> **Note**:
> The first encountered `ADD` instruction will invalidate the cache for all
> following instructions from the Dockerfile if the contents of `<src>` have
> changed. This includes invalidating the cache for `RUN` instructions.
> See the [`Dockerfile` Best Practices
guide](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/#/build-cache) for more information.


`ADD` obeys the following rules:

- The `<src>` path must be inside the *context* of the build;
  you cannot `ADD ../something /something`, because the first step of a
  `docker build` is to send the context directory (and subdirectories) to the
  docker daemon.

- If `<src>` is a URL and `<dest>` does not end with a trailing slash, then a
  file is downloaded from the URL and copied to `<dest>`.

- If `<src>` is a URL and `<dest>` does end with a trailing slash, then the
  filename is inferred from the URL and the file is downloaded to
  `<dest>/<filename>`. For instance, `ADD http://example.com/foobar /` would
  create the file `/foobar`. The URL must have a nontrivial path so that an
  appropriate filename can be discovered in this case (`http://example.com`
  will not work).

- If `<src>` is a directory, the entire contents of the directory are copied,
  including filesystem metadata.

> **Note**:
> The directory itself is not copied, just its contents.

- If `<src>` is a *local* tar archive in a recognized compression format
  (identity, gzip, bzip2 or xz) then it is unpacked as a directory. Resources
  from *remote* URLs are **not** decompressed. When a directory is copied or
  unpacked, it has the same behavior as `tar -x`, the result is the union of:

    1. Whatever existed at the destination path and
    2. The contents of the source tree, with conflicts resolved in favor
       of "2." on a file-by-file basis.

  > **Note**:
  > Whether a file is identified as a recognized compression format or not
  > is done solely based on the contents of the file, not the name of the file.
  > For example, if an empty file happens to end with `.tar.gz` this will not
  > be recognized as a compressed file and **will not** generate any kind of
  > decompression error message, rather the file will simply be copied to the
  > destination.

- If `<src>` is any other kind of file, it is copied individually along with
  its metadata. In this case, if `<dest>` ends with a trailing slash `/`, it
  will be considered a directory and the contents of `<src>` will be written
  at `<dest>/base(<src>)`.

- If multiple `<src>` resources are specified, either directly or due to the
  use of a wildcard, then `<dest>` must be a directory, and it must end with
  a slash `/`.

- If `<dest>` does not end with a trailing slash, it will be considered a
  regular file and the contents of `<src>` will be written at `<dest>`.

- If `<dest>` doesn't exist, it is created along with all missing directories
  in its path.

## COPY

COPY has two forms:

- `COPY [--chown=<user>:<group>] <src>... <dest>`
- `COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]` (this form is required for paths containing
whitespace)

> **Note**:
> The `--chown` feature is only supported on Dockerfiles used to build Linux containers,
> and will not work on Windows containers. Since user and group ownership concepts do
> not translate between Linux and Windows, the use of `/etc/passwd` and `/etc/group` for
> translating user and group names to IDs restricts this feature to only be viable for
> for Linux OS-based containers.

The `COPY` instruction copies new files or directories from `<src>`
and adds them to the filesystem of the container at the path `<dest>`.

Multiple `<src>` resources may be specified but the paths of files and
directories will be interpreted as relative to the source of the context
of the build.

Each `<src>` may contain wildcards and matching will be done using Go's
[filepath.Match](http://golang.org/pkg/path/filepath#Match) rules. For example:

    COPY hom* /mydir/        # adds all files starting with "hom"
    COPY hom?.txt /mydir/    # ? is replaced with any single character, e.g., "home.txt"

The `<dest>` is an absolute path, or a path relative to `WORKDIR`, into which
the source will be copied inside the destination container.

    COPY test relativeDir/   # adds "test" to `WORKDIR`/relativeDir/
    COPY test /absoluteDir/  # adds "test" to /absoluteDir/


When copying files or directories that contain special characters (such as `[`
and `]`), you need to escape those paths following the Golang rules to prevent
them from being treated as a matching pattern. For example, to copy a file
named `arr[0].txt`, use the following;

    COPY arr[[]0].txt /mydir/    # copy a file named "arr[0].txt" to /mydir/

All new files and directories are created with a UID and GID of 0, unless the
optional `--chown` flag specifies a given username, groupname, or UID/GID
combination to request specific ownership of the copied content. The
format of the `--chown` flag allows for either username and groupname strings
or direct integer UID and GID in any combination. Providing a username without
groupname or a UID without GID will use the same numeric UID as the GID. If a
username or groupname is provided, the container's root filesystem
`/etc/passwd` and `/etc/group` files will be used to perform the translation
from name to integer UID or GID respectively. The following examples show
valid definitions for the `--chown` flag:

    COPY --chown=55:mygroup files* /somedir/
    COPY --chown=bin files* /somedir/
    COPY --chown=1 files* /somedir/
    COPY --chown=10:11 files* /somedir/

If the container root filesystem does not contain either `/etc/passwd` or
`/etc/group` files and either user or group names are used in the `--chown`
flag, the build will fail on the `COPY` operation. Using numeric IDs requires
no lookup and will not depend on container root filesystem content.

> **Note**:
> If you build using STDIN (`docker build - < somefile`), there is no
> build context, so `COPY` can't be used.

Optionally `COPY` accepts a flag `--from=<name|index>` that can be used to set
the source location to a previous build stage (created with `FROM .. AS <name>`)
that will be used instead of a build context sent by the user. The flag also
accepts a numeric index assigned for all previous build stages started with
`FROM` instruction. In case a build stage with a specified name can't be found an
image with the same name is attempted to be used instead.

`COPY` obeys the following rules:

- The `<src>` path must be inside the *context* of the build;
  you cannot `COPY ../something /something`, because the first step of a
  `docker build` is to send the context directory (and subdirectories) to the
  docker daemon.

- If `<src>` is a directory, the entire contents of the directory are copied,
  including filesystem metadata.

> **Note**:
> The directory itself is not copied, just its contents.

- If `<src>` is any other kind of file, it is copied individually along with
  its metadata. In this case, if `<dest>` ends with a trailing slash `/`, it
  will be considered a directory and the contents of `<src>` will be written
  at `<dest>/base(<src>)`.

- If multiple `<src>` resources are specified, either directly or due to the
  use of a wildcard, then `<dest>` must be a directory, and it must end with
  a slash `/`.

- If `<dest>` does not end with a trailing slash, it will be considered a
  regular file and the contents of `<src>` will be written at `<dest>`.

- If `<dest>` doesn't exist, it is created along with all missing directories
  in its path.

## ENTRYPOINT

ENTRYPOINT has two forms:

- `ENTRYPOINT ["executable", "param1", "param2"]`
  (*exec* form, preferred)
- `ENTRYPOINT command param1 param2`
  (*shell* form)

An `ENTRYPOINT` allows you to configure a container that will run as an executable.

For example, the following will start nginx with its default content, listening
on port 80:

    docker run -i -t --rm -p 80:80 nginx

Command line arguments to `docker run <image>` will be appended after all
elements in an *exec* form `ENTRYPOINT`, and will override all elements specified
using `CMD`.
This allows arguments to be passed to the entry point, i.e., `docker run <image> -d`
will pass the `-d` argument to the entry point.
You can override the `ENTRYPOINT` instruction using the `docker run --entrypoint`
flag.

The *shell* form prevents any `CMD` or `run` command line arguments from being
used, but has the disadvantage that your `ENTRYPOINT` will be started as a
subcommand of `/bin/sh -c`, which does not pass signals.
This means that the executable will not be the container's `PID 1` - and
will _not_ receive Unix signals - so your executable will not receive a
`SIGTERM` from `docker stop <container>`.

Only the last `ENTRYPOINT` instruction in the `Dockerfile` will have an effect.

### Exec form ENTRYPOINT example

You can use the *exec* form of `ENTRYPOINT` to set fairly stable default commands
and arguments and then use either form of `CMD` to set additional defaults that
are more likely to be changed.

    FROM ubuntu
    ENTRYPOINT ["top", "-b"]
    CMD ["-c"]

When you run the container, you can see that `top` is the only process:

    $ docker run -it --rm --name test  top -H
    top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
    Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
    KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

      PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
        1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top

To examine the result further, you can use `docker exec`:

    $ docker exec -it test ps aux
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  2.6  0.1  19752  2352 ?        Ss+  08:24   0:00 top -b -H
    root         7  0.0  0.1  15572  2164 ?        R+   08:25   0:00 ps aux

And you can gracefully request `top` to shut down using `docker stop test`.

The following `Dockerfile` shows using the `ENTRYPOINT` to run Apache in the
foreground (i.e., as `PID 1`):

```
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

If you need to write a starter script for a single executable, you can ensure that
the final executable receives the Unix signals by using `exec` and `gosu`
commands:

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

Lastly, if you need to do some extra cleanup (or communicate with other containers)
on shutdown, or are co-ordinating more than one executable, you may need to ensure
that the `ENTRYPOINT` script receives the Unix signals, passes them on, and then
does some more work:

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

If you run this image with `docker run -it --rm -p 80:80 --name test apache`,
you can then examine the container's processes with `docker exec`, or `docker top`,
and then ask the script to stop Apache:

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

> **Note:** you can override the `ENTRYPOINT` setting using `--entrypoint`,
> but this can only set the binary to *exec* (no `sh -c` will be used).

> **Note**:
> The *exec* form is parsed as a JSON array, which means that
> you must use double-quotes (") around words not single-quotes (').

> **Note**:
> Unlike the *shell* form, the *exec* form does not invoke a command shell.
> This means that normal shell processing does not happen. For example,
> `ENTRYPOINT [ "echo", "$HOME" ]` will not do variable substitution on `$HOME`.
> If you want shell processing then either use the *shell* form or execute
> a shell directly, for example: `ENTRYPOINT [ "sh", "-c", "echo $HOME" ]`.
> When using the exec form and executing a shell directly, as in the case for
> the shell form, it is the shell that is doing the environment variable
> expansion, not docker.

### Shell form ENTRYPOINT example

You can specify a plain string for the `ENTRYPOINT` and it will execute in `/bin/sh -c`.
This form will use shell processing to substitute shell environment variables,
and will ignore any `CMD` or `docker run` command line arguments.
To ensure that `docker stop` will signal any long running `ENTRYPOINT` executable
correctly, you need to remember to start it with `exec`:

    FROM ubuntu
    ENTRYPOINT exec top -b

When you run this image, you'll see the single `PID 1` process:

    $ docker run -it --rm --name test top
    Mem: 1704520K used, 352148K free, 0K shrd, 0K buff, 140368121167873K cached
    CPU:   5% usr   0% sys   0% nic  94% idle   0% io   0% irq   0% sirq
    Load average: 0.08 0.03 0.05 2/98 6
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     R     3164   0%   0% top -b

Which will exit cleanly on `docker stop`:

    $ /usr/bin/time docker stop test
    test
    real	0m 0.20s
    user	0m 0.02s
    sys	0m 0.04s

If you forget to add `exec` to the beginning of your `ENTRYPOINT`:

    FROM ubuntu
    ENTRYPOINT top -b
    CMD --ignored-param1

You can then run it (giving it a name for the next step):

    $ docker run -it --name test top --ignored-param2
    Mem: 1704184K used, 352484K free, 0K shrd, 0K buff, 140621524238337K cached
    CPU:   9% usr   2% sys   0% nic  88% idle   0% io   0% irq   0% sirq
    Load average: 0.01 0.02 0.05 2/101 7
      PID  PPID USER     STAT   VSZ %VSZ %CPU COMMAND
        1     0 root     S     3168   0%   0% /bin/sh -c top -b cmd cmd2
        7     1 root     R     3164   0%   0% top -b

You can see from the output of `top` that the specified `ENTRYPOINT` is not `PID 1`.

If you then run `docker stop test`, the container will not exit cleanly - the
`stop` command will be forced to send a `SIGKILL` after the timeout:

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

### Understand how CMD and ENTRYPOINT interact

Both `CMD` and `ENTRYPOINT` instructions define what command gets executed when running a container.
There are few rules that describe their co-operation.

1. Dockerfile should specify at least one of `CMD` or `ENTRYPOINT` commands.

2. `ENTRYPOINT` should be defined when using the container as an executable.

3. `CMD` should be used as a way of defining default arguments for an `ENTRYPOINT` command
or for executing an ad-hoc command in a container.

4. `CMD` will be overridden when running the container with alternative arguments.

The table below shows what command is executed for different `ENTRYPOINT` / `CMD` combinations:

|                                | No ENTRYPOINT              | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT ["exec_entry", "p1_entry"]          |
|:-------------------------------|:---------------------------|:-------------------------------|:-----------------------------------------------|
| **No CMD**                     | *error, not allowed*       | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry                            |
| **CMD ["exec_cmd", "p1_cmd"]** | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd            |
| **CMD ["p1_cmd", "p2_cmd"]**   | p1_cmd p2_cmd              | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd              |
| **CMD exec_cmd p1_cmd**        | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |

## VOLUME

    VOLUME ["/data"]

The `VOLUME` instruction creates a mount point with the specified name
and marks it as holding externally mounted volumes from native host or other
containers. The value can be a JSON array, `VOLUME ["/var/log/"]`, or a plain
string with multiple arguments, such as `VOLUME /var/log` or `VOLUME /var/log
/var/db`. For more information/examples and mounting instructions via the
Docker client, refer to
[*Share Directories via Volumes*](https://docs.docker.com/engine/tutorials/dockervolumes/#/mount-a-host-directory-as-a-data-volume)
documentation.

The `docker run` command initializes the newly created volume with any data
that exists at the specified location within the base image. For example,
consider the following Dockerfile snippet:

    FROM ubuntu
    RUN mkdir /myvol
    RUN echo "hello world" > /myvol/greeting
    VOLUME /myvol

This Dockerfile results in an image that causes `docker run` to
create a new mount point at `/myvol` and copy the  `greeting` file
into the newly created volume.

### Notes about specifying volumes

Keep the following things in mind about volumes in the `Dockerfile`.

- **Volumes on Windows-based containers**: When using Windows-based containers,
  the destination of a volume inside the container must be one of:

  - a non-existing or empty directory
  - a drive other than `C:`

- **Changing the volume from within the Dockerfile**: If any build steps change the
  data within the volume after it has been declared, those changes will be discarded.

- **JSON formatting**: The list is parsed as a JSON array.
  You must enclose words with double quotes (`"`)rather than single quotes (`'`).

- **The host directory is declared at container run-time**: The host directory
  (the mountpoint) is, by its nature, host-dependent. This is to preserve image
  portability, since a given host directory can't be guaranteed to be available
  on all hosts. For this reason, you can't mount a host directory from
  within the Dockerfile. The `VOLUME` instruction does not support specifying a `host-dir`
  parameter.  You must specify the mountpoint when you create or run the container.

## USER

    USER <user>[:<group>]
or
    USER <UID>[:<GID>]

The `USER` instruction sets the user name (or UID) and optionally the user
group (or GID) to use when running the image and for any `RUN`, `CMD` and
`ENTRYPOINT` instructions that follow it in the `Dockerfile`.

> **Warning**:
> When the user doesn't have a primary group then the image (or the next
> instructions) will be run with the `root` group.

> On Windows, the user must be created first if it's not a built-in account.
> This can be done with the `net user` command called as part of a Dockerfile.

```Dockerfile
    FROM microsoft/windowsservercore
    # Create Windows user in the container
    RUN net user /add patrick
    # Set it for subsequent commands
    USER patrick
```


## WORKDIR

    WORKDIR /path/to/workdir

The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`,
`ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the `Dockerfile`.
If the `WORKDIR` doesn't exist, it will be created even if it's not used in any
subsequent `Dockerfile` instruction.

The `WORKDIR` instruction can be used multiple times in a `Dockerfile`. If a
relative path is provided, it will be relative to the path of the previous
`WORKDIR` instruction. For example:

    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd

The output of the final `pwd` command in this `Dockerfile` would be
`/a/b/c`.

The `WORKDIR` instruction can resolve environment variables previously set using
`ENV`. You can only use environment variables explicitly set in the `Dockerfile`.
For example:

    ENV DIRPATH /path
    WORKDIR $DIRPATH/$DIRNAME
    RUN pwd

The output of the final `pwd` command in this `Dockerfile` would be
`/path/$DIRNAME`

## ARG

    ARG <name>[=<default value>]

The `ARG` instruction defines a variable that users can pass at build-time to
the builder with the `docker build` command using the `--build-arg <varname>=<value>`
flag. If a user specifies a build argument that was not
defined in the Dockerfile, the build outputs a warning.

```
[Warning] One or more build-args [foo] were not consumed.
```

A Dockerfile may include one or more `ARG` instructions. For example,
the following is a valid Dockerfile:

```
FROM busybox
ARG user1
ARG buildno
...
```

> **Warning:** It is not recommended to use build-time variables for
>  passing secrets like github keys, user credentials etc. Build-time variable
>  values are visible to any user of the image with the `docker history` command.

### Default values

An `ARG` instruction can optionally include a default value:

```
FROM busybox
ARG user1=someuser
ARG buildno=1
...
```

If an `ARG` instruction has a default value and if there is no value passed
at build-time, the builder uses the default.

### Scope

An `ARG` variable definition comes into effect from the line on which it is
defined in the `Dockerfile` not from the argument's use on the command-line or
elsewhere.  For example, consider this Dockerfile:

```
1 FROM busybox
2 USER ${user:-some_user}
3 ARG user
4 USER $user
...
```
A user builds this file by calling:

```
$ docker build --build-arg user=what_user .
```

The `USER` at line 2 evaluates to `some_user` as the `user` variable is defined on the
subsequent line 3. The `USER` at line 4 evaluates to `what_user` as `user` is
defined and the `what_user` value was passed on the command line. Prior to its definition by an
`ARG` instruction, any use of a variable results in an empty string.

An `ARG` instruction goes out of scope at the end of the build
stage where it was defined. To use an arg in multiple stages, each stage must
include the `ARG` instruction.

```
FROM busybox
ARG SETTINGS
RUN ./run/setup $SETTINGS

FROM busybox
ARG SETTINGS
RUN ./run/other $SETTINGS
```

### Using ARG variables

You can use an `ARG` or an `ENV` instruction to specify variables that are
available to the `RUN` instruction. Environment variables defined using the
`ENV` instruction always override an `ARG` instruction of the same name. Consider
this Dockerfile with an `ENV` and `ARG` instruction.

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER v1.0.0
4 RUN echo $CONT_IMG_VER
```
Then, assume this image is built with this command:

```
$ docker build --build-arg CONT_IMG_VER=v2.0.1 .
```

In this case, the `RUN` instruction uses `v1.0.0` instead of the `ARG` setting
passed by the user:`v2.0.1` This behavior is similar to a shell
script where a locally scoped variable overrides the variables passed as
arguments or inherited from environment, from its point of definition.

Using the example above but a different `ENV` specification you can create more
useful interactions between `ARG` and `ENV` instructions:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER ${CONT_IMG_VER:-v1.0.0}
4 RUN echo $CONT_IMG_VER
```

Unlike an `ARG` instruction, `ENV` values are always persisted in the built
image. Consider a docker build without the `--build-arg` flag:

```
$ docker build .
```

Using this Dockerfile example, `CONT_IMG_VER` is still persisted in the image but
its value would be `v1.0.0` as it is the default set in line 3 by the `ENV` instruction.

The variable expansion technique in this example allows you to pass arguments
from the command line and persist them in the final image by leveraging the
`ENV` instruction. Variable expansion is only supported for [a limited set of
Dockerfile instructions.](#environment-replacement)

### Predefined ARGs

Docker has a set of predefined `ARG` variables that you can use without a
corresponding `ARG` instruction in the Dockerfile.

* `HTTP_PROXY`
* `http_proxy`
* `HTTPS_PROXY`
* `https_proxy`
* `FTP_PROXY`
* `ftp_proxy`
* `NO_PROXY`
* `no_proxy`

To use these, simply pass them on the command line using the flag:

```
--build-arg <varname>=<value>
```

By default, these pre-defined variables are excluded from the output of
`docker history`. Excluding them reduces the risk of accidentally leaking
sensitive authentication information in an `HTTP_PROXY` variable.

For example, consider building the following Dockerfile using
`--build-arg HTTP_PROXY=http://user:pass@proxy.lon.example.com`

``` Dockerfile
FROM ubuntu
RUN echo "Hello World"
```

In this case, the value of the `HTTP_PROXY` variable is not available in the
`docker history` and is not cached. If you were to change location, and your
proxy server changed to `http://user:pass@proxy.sfo.example.com`, a subsequent
build does not result in a cache miss.

If you need to override this behaviour then you may do so by adding an `ARG`
statement in the Dockerfile as follows:

``` Dockerfile
FROM ubuntu
ARG HTTP_PROXY
RUN echo "Hello World"
```

When building this Dockerfile, the `HTTP_PROXY` is preserved in the
`docker history`, and changing its value invalidates the build cache.

### Impact on build caching

`ARG` variables are not persisted into the built image as `ENV` variables are.
However, `ARG` variables do impact the build cache in similar ways. If a
Dockerfile defines an `ARG` variable whose value is different from a previous
build, then a "cache miss" occurs upon its first usage, not its definition. In
particular, all `RUN` instructions following an `ARG` instruction use the `ARG`
variable implicitly (as an environment variable), thus can cause a cache miss.
All predefined `ARG` variables are exempt from caching unless there is a
matching `ARG` statement in the `Dockerfile`.

For example, consider these two Dockerfile:

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

If you specify `--build-arg CONT_IMG_VER=<value>` on the command line, in both
cases, the specification on line 2 does not cause a cache miss; line 3 does
cause a cache miss.`ARG CONT_IMG_VER` causes the RUN line to be identified
as the same as running `CONT_IMG_VER=<value>` echo hello, so if the `<value>`
changes, we get a cache miss.

Consider another example under the same command line:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER $CONT_IMG_VER
4 RUN echo $CONT_IMG_VER
```
In this example, the cache miss occurs on line 3. The miss happens because
the variable's value in the `ENV` references the `ARG` variable and that
variable is changed through the command line. In this example, the `ENV`
command causes the image to include the value.

If an `ENV` instruction overrides an `ARG` instruction of the same name, like
this Dockerfile:

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER hello
4 RUN echo $CONT_IMG_VER
```

Line 3 does not cause a cache miss because the value of `CONT_IMG_VER` is a
constant (`hello`). As a result, the environment variables and values used on
the `RUN` (line 4) doesn't change between builds.

## ONBUILD

    ONBUILD [INSTRUCTION]

The `ONBUILD` instruction adds to the image a *trigger* instruction to
be executed at a later time, when the image is used as the base for
another build. The trigger will be executed in the context of the
downstream build, as if it had been inserted immediately after the
`FROM` instruction in the downstream `Dockerfile`.

Any build instruction can be registered as a trigger.

This is useful if you are building an image which will be used as a base
to build other images, for example an application build environment or a
daemon which may be customized with user-specific configuration.

For example, if your image is a reusable Python application builder, it
will require application source code to be added in a particular
directory, and it might require a build script to be called *after*
that. You can't just call `ADD` and `RUN` now, because you don't yet
have access to the application source code, and it will be different for
each application build. You could simply provide application developers
with a boilerplate `Dockerfile` to copy-paste into their application, but
that is inefficient, error-prone and difficult to update because it
mixes with application-specific code.

The solution is to use `ONBUILD` to register advance instructions to
run later, during the next build stage.

Here's how it works:

1. When it encounters an `ONBUILD` instruction, the builder adds a
   trigger to the metadata of the image being built. The instruction
   does not otherwise affect the current build.
2. At the end of the build, a list of all triggers is stored in the
   image manifest, under the key `OnBuild`. They can be inspected with
   the `docker inspect` command.
3. Later the image may be used as a base for a new build, using the
   `FROM` instruction. As part of processing the `FROM` instruction,
   the downstream builder looks for `ONBUILD` triggers, and executes
   them in the same order they were registered. If any of the triggers
   fail, the `FROM` instruction is aborted which in turn causes the
   build to fail. If all triggers succeed, the `FROM` instruction
   completes and the build continues as usual.
4. Triggers are cleared from the final image after being executed. In
   other words they are not inherited by "grand-children" builds.

For example you might add something like this:

    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]

> **Warning**: Chaining `ONBUILD` instructions using `ONBUILD ONBUILD` isn't allowed.

> **Warning**: The `ONBUILD` instruction may not trigger `FROM` or `MAINTAINER` instructions.

## STOPSIGNAL

    STOPSIGNAL signal

The `STOPSIGNAL` instruction sets the system call signal that will be sent to the container to exit.
This signal can be a valid unsigned number that matches a position in the kernel's syscall table, for instance 9,
or a signal name in the format SIGNAME, for instance SIGKILL.

## HEALTHCHECK

The `HEALTHCHECK` instruction has two forms:

* `HEALTHCHECK [OPTIONS] CMD command` (check container health by running a command inside the container)
* `HEALTHCHECK NONE` (disable any healthcheck inherited from the base image)

The `HEALTHCHECK` instruction tells Docker how to test a container to check that
it is still working. This can detect cases such as a web server that is stuck in
an infinite loop and unable to handle new connections, even though the server
process is still running.

When a container has a healthcheck specified, it has a _health status_ in
addition to its normal status. This status is initially `starting`. Whenever a
health check passes, it becomes `healthy` (whatever state it was previously in).
After a certain number of consecutive failures, it becomes `unhealthy`.

The options that can appear before `CMD` are:

* `--interval=DURATION` (default: `30s`)
* `--timeout=DURATION` (default: `30s`)
* `--start-period=DURATION` (default: `0s`)
* `--retries=N` (default: `3`)

The health check will first run **interval** seconds after the container is
started, and then again **interval** seconds after each previous check completes.

If a single run of the check takes longer than **timeout** seconds then the check
is considered to have failed.

It takes **retries** consecutive failures of the health check for the container
to be considered `unhealthy`.

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
        
