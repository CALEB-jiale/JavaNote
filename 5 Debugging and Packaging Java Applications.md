# Debugging and Packaging Java Applications

## Types of Errors

1. Compile-time Errors: 语法错误导致的错误，编译器能检测到
2. Run-time Errors: 逻辑错误等，用 Debugger 来检测

## Debugging

1. 在命令行前点击增加断点（或 Run -> ViewBreakpoints 或 快捷键 command + shift + F8）
2. Debug 运行（Control + D）

调用栈

## Packaging

构建一个 Java 程序后，若想分享给别人使用， 需要将代码打包进一个 jar 文件中，jar 是 Java Archive 的简称，他是一个包文件形式，包含用于分享的所有代码。

构建方式：

1. File -> Project Structure -> Artifacts -> + -> JAR -> From modules with dependencies
2. 选择 Main Class
3. OK and OK
4. Build -> Build Artifacts
5. 选择 Build，Rebuild 或者 Clean，第一次选择 Build
6. 在代码左侧的 Project 面板中找到 out 文件夹，里面有 Artifacts，一路顺下去可找到 jar 文件
7. 右击并选择在终端打开 jar 文件
8. 利用指令 `java -jar filename.jar` 来运行代码

