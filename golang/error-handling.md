## Errors Handling  [Return Parent](../README.md)

- #### Handling panics

  Two built-in functions, `panic` and `recover`, assist in reporting and handling [run-time panics](https://golang.org/ref/spec#Run_time_panics) and program-defined error conditions.

  如果在函数中显式的调用panic函数，或者执行触发运行期的panic。则该函数（设函数F)内由defer 调用的函数将会执行，然后调用函数F内由defer 调用的函数将会被执行，如此依次执行上级函数内由defer 调用的函数。随后，程序崩溃并输出日志信息，日志信息包括 panic value 和函数调用的堆栈跟踪信息。

  The `recover` function allows a program to manage behavior of a panicking goroutine. Suppose a function `G` defers a function `D` that calls `recover` and a panic occurs in a function on the same goroutine in which `G` is executing. When the running of deferred functions reaches `D`, the return value of `D`'s call to `recover` will be the value passed to the call of `panic`. If `D` returns normally, without starting a new `panic`, the panicking sequence stops. In that case, the state of functions called between `G` and the call to `panic` is discarded, and normal execution resumes. **Any functions deferred by `G` before `D` are then run and `G`'s execution terminates by returning to its caller.**换句话说，在D之前被deferred的函数还是会被执行的。**那不在是G内被deferred的函数呢？会被执行吗？**

- 

- 