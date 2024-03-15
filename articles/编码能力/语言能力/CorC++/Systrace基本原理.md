---
title: Systrace基本原理
date: 2023-12-25
---
# Systrace基本原理

## 背景

性能分析对于应用开发极为重要。作为一个经久不衰的操作系统，Linux 提供了追踪分析（Tracing）方法。这种方法利用内核中的各种追踪机制来收集系统运行的详细信息。通过追踪分析，开发者可以收集包括函数调用、系统调用、CPU 调度事件等在内的数据，这有助于识别性能瓶颈。在此过程中，Ftrace 作为一项重要工具，利用内核中的追踪点（tracepoints）和其他机制（如函数追踪器、事件追踪器等）来搜集数据，主要用于剖析 Linux 内核的性能和行为。

在 Android 系统中，ATrace 允许开发者通过在应用程序代码中添加特定的 API 调用来启用追踪点，并在运行时收集性能数据。这种方法主要用于分析 Android 应用程序的性能，如 UI 渲染、线程活动、响应时间等。

Systrace 是 Android4.1 中新增的性能数据采样和分析工具，它结合了 Ftrace 和 ATrace 的功能。它使用 Ftrace 来收集系统级的性能数据，收集 Android 关键子系统的运行信息，并支持使用 ATrace 插入的应用级追踪点，帮助开发者全面直观的分析系统情况，改进性能。

## 组成

Systrace 的构成包括三大部分：

- **Ftrace 部分**：作为 Linux 内核的追踪框架，负责搜集内核层面的事件数据，例如 CPU 调度、中断处理和系统调用等。
- **Atrace 部分**：专为 Android 平台设计的追踪工具，允许开发者在应用代码中设置追踪点，用于搜集应用运行时的行为数据。
- **数据分析**：提供一个图形化界面，整合并展示这些数据，形成时间线视图，协助开发者理解应用程序与系统级别性能的相互作用。

## 使用

1. **添加应用自定义追踪点（可选）**

   在您希望监测性能的代码部分添加自定义追踪点：

   ```
   kotlinCopy codeimport android.os.Trace
   
   fun traceMethod() {
       Trace.beginSection("traceMethodTrace") // 在 Java 中代码相同
       // ...实际代码...
       Trace.endSection()
   }
   ```

2. **开启 Systrace**

   当需要进行性能分析时，使用以下脚本开启 Systrace：

   ```
   bashCopy code
   python2 ./systrace.py -a com.DefaultCompany.modifytest
   ```

   注意事项：

   - 使用 Python 2 运行脚本。
   - platform-tools 版本 33.0.0 是最后一个包含 Systrace 的版本。
   - 不使用 `-a` 参数将导致 Trace 中缺少自定义追踪点。

3. **运行应用并收集数据**

   在 Systrace 运行期间，正常使用您的应用以收集性能数据。

4. **生成 Systrace 报告**

   应用运行结束后，您将得到一个 Systrace 性能报告。

5. **分析性能**

   访问 [Perfetto UI](https://ui.perfetto.dev/) 并上传 Systrace 报告以分析应用的性能表现。

## 关于OpenHarmony

OpenHarmony 提供了 HiTrace，它是与 Systrace 相对应的工具，用于执行类似的性能追踪操作。

1. **添加应用自定义追踪点（可选）:**

   - **ArkTS**:

     ```tsx
     import hitrace from '@ohos.hiTraceMeter';
     
     function traceMethod() {
         hitrace.startTrace("TraceName", 1001);
         //...实际代码...
         hitrace.finishTrace("TraceName", 1001); // 参数应与 startTrace 中的对应
     }
     ```

   - **Native C++**:

     ```c++
     #include <hitrace/trace.h>
     
     void traceMethod() {
         OH_Hitrace_StartTrace("TraceName");
         //...实际代码...
         OH_Hitrace_FinishTrace();
     }
     ```

2. **启动追踪**：

   连接设备后，打开 HDC（HarmonyOS Device Connect）命令行界面，执行以下命令以开始追踪：

   ```shell
   hitrace --trace_begin -b 204800 app
   ```

   -b参数提高缓存容量，否则会漏tracepoint造成trace嵌套。

3. **查看结果**：

   使用以下命令导出追踪结果：

   ```shell
   hitrace --trace_dump -o /data/local/tmp/test.ftrace
   ```

4. **提取追踪文件**：

   使用 HDC 工具从设备获取追踪文件：

   ```shell
   hdc file recv /data/local/tmp/test.ftrace test.ftrace
   ```

5. **分析性能**

​		访问 [Perfetto UI](https://ui.perfetto.dev/) 并上传 Hitrace 报告以分析应用的性能表现。