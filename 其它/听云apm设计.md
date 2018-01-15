# 服务器端

## 概览
提供服务器端整体概览

-----> 应用概览
提供应用的apdex和错误率

### 关键事务概览
关键事务的概览：相应时间、调用次数、错误率

### 


# 浏览器端

+-- _config.yml
+-- _drafts
|   +-- begin-with-the-crazy-ideas.textile
|   +-- on-simplicity-in-technology.markdown


附录：
* Apdex
Apdex 定义了应用响应时间的最优门槛为 T，另外根据应用响应时间结合 T 定义了三种不同的性能表现：
Satisfied（满意）：应用响应时间低于或等于 T（T 由性能评估人员根据预期性能要求确定），比如 T 为 1.5s，则一个耗时 1s 的响应结果则可以认为是 satisfied 的。
Tolerating（可容忍）：应用响应时间大于 T，但同时小于或等于 4T。假设应用设定的 T 值为 1s，则 4 * 1 = 4 秒极为应用响应时间的容忍上限。
Frustrated（烦躁期）：应用响应时间大于 4T。
Apdex<sub>t</sub> = (Satisfied Count + Tolerating Count / 2) / Total Samples
其中 Satisfied Count 就是指定采样时间内响应时间满足 Satisfied 要求的应用响应次数；而 Tolerating Count 就是指定采样时间内响应时间满足
Tolerating 要求的应用响应次数；最后的 Total Samples 就是总的采样次数总数。从公式可以看出，应用的 Apdex 得分与采样持续时间无关，
与目标响应时间 T 相关（在采用总数固定的情况下，T 通过影响 Satisfied Count以及 Tolerating Count的值间接影响最终的得分）。
举例来说，假设你的应用期待的响应时间能够在 1000 ms 内，在 100 次采样中，有 50 次应用响应时间低于 1000 ms，30 次应用响应时间处于
 1000 ms 到 4000 ms（ 4 * 1000ms） 之间，剩下 20 次响应时间长于 4000 ms，那么，该应用在 T = 1000ms 的情况下的 Apdex 值为：
(50 + 30 / 2) / 100 = 0.65

* 错误率