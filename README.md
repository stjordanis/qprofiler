# qprofiler
q/kdb+ function profiler is a production safe profiler that can be used for live debugging and performance troubleshooting.

# Usage

## Profiling
To use the profiler, first load the P.q file.  Then for any function execution you wish to debug, simply prepend P) to the execution.

```
// First we define a function for testing
f:{[x] a:x%5; b:sin each x; a+b}

// Then we run the function through P) to see the status of each line's execution
q)P)f[til 1000]
i pct  time                 x             w
--------------------------------------------
0                           [x]           ()
1 3.8  0D00:00:00.000070000 a:x%5;        ::
2 95.1 0D00:00:00.001763000 b:sin each x; ::
3 0.9  0D00:00:00.000016000 a+b           ::

```

Note above the columns indicates the elapsed time for each line of execution, and percentage of time that line took in the overall function's run time.  This would help us to narrow down to performance issues quicklly.


```
// Lets redefine a function for without each
f:{[x] a:x%5; b:sin x; a+b}

// Then we run the function through P) again to see the change
q)P)f[til 1000]
i pct  time                 x        w
---------------------------------------
0                           [x]      ()
1 26   0D00:00:00.000039000 a:x%5;   ::
2 58.7 0D00:00:00.000088000 b:sin x; ::
3 9.3  0D00:00:00.000014000 a+b      ::

```


## Error tracking

The profiler would stop execution when it encounters an error

```
// Lets redefine the function to introduce a line that would error
f:{[x] a:x%5; b:sin x; a:`$string a; c:a+b; :c}

// Then we can run it through P) to find the offending line
q)P)f[til 3]
Function errored with 'type
i pct  time                 x             w
--------------------------------------------
0                           [x]           ()
1 15.8 0D00:00:00.000006000 a:x%5;        ::
2 10.5 0D00:00:00.000004000 b:sin x;      ::
3 34.2 0D00:00:00.000013000 a:`$string a; ::
4 34.2 0D00:00:00.000013000 c:a+b;        ::
5                           :c            ()
```


The profiler could even be used to track variable changes from line to line with // at the end

```
// Track just one variable
q)P)f[til 3]//a
Function errored with 'type
i pct  time                 x             w
----------------------------------------------------
0                           [x]           ()
1 14.3 0D00:00:00.000009000 a:x%5;        0 0.2 0.4
2 14.3 0D00:00:00.000009000 b:sin x;      0 0.2 0.4
3 28.6 0D00:00:00.000018000 a:`$string a; `0`0.2`0.4
4 38.1 0D00:00:00.000024000 c:a+b;        ::
5                           :c            ()

// You can access the variable with .P.w with the line number to get the result of execution after the line.  e.g.
q).P.w 3

// Further, one can track multiple variables with
P)f[til 3]//(a;b)

// But note all the intemediate variables are stored in memory thus remember to track smaller variables only

```

The profiling result are saved in .P.x, and the results are in array .P.w


## Special notes
There're a few known issue with the profiler:
- It does not trancende into if and $ blocks, and treat them as one line
- The text after P) must wrap parameters in one and only one pair of enclosing [].  When the function is arity of 0 (no arguments), one need to call it with P)f[::]
- Line number in column i could be shifted if the underlying function has no parameter definition



## Credit
Credit goes to Rmorris and Avrabecz 








