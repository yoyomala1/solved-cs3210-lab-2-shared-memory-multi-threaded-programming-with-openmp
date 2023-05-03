Download Link: https://assignmentchef.com/product/solved-cs3210-lab-2-shared-memory-multi-threaded-programming-with-openmp
<br>
<h2>Introducing the Problem Scenario: Matrix Multiplication</h2>

In lecture, we (very) briefly mention the matrix multiplication problem. This lab is based on the different ways of parallelizing this problem.

Given an <strong>n x m matrix A </strong>and an <strong>m x p matrix B</strong>, the product of the two matrices <strong>(AB) </strong>is an <strong>n x p </strong>matrix with entries defined by:

A straightforward sequential translation is given below (Sequential: mm-seq.c):

void mm( matrix a, matrix b, matrix result )

{

int i, j, k;

//assuming square matrices of (size x size)

for (i = 0; i &lt; size; i++)

for (j = 0; j &lt; size; j++) for (k = 0; k &lt; size; k++) result[i][j] += a[i][k] * b[k][j];

}

1

• Compile the code in a terminal (console):

&gt; gcc -o mm0 mm-seq.c

<ul>

 <li>Run the program in a terminal (console):</li>

</ul>

&gt; ./mm0 10

<table width="601">

 <tbody>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 1</u></strong>Compile and run mm-seq.c. Run with matrix sizes in the range of 1000+ to get a longer execution time.</td>

  </tr>

 </tbody>

</table>

<h2>Shared-Memory OpenMP Programs</h2>

One quick way to parallelize this problem is to create task to handle each row in the result matrix. As each row can be calculated independently, there is no need to worry about synchronization. Also, if the content of the matrices are shared between the tasks, there is no addition communication needed! We make use of this idea and translate it into an OpenMP program as given below (OpenMP: mm-omp.c):

void mm(matrix a, matrix b, matrix result)

{

int i, j, k;

// Parallelize the multiplication

// Each thread will work on one iteration of the

// outer-most loop

// Variables (a, b, result) are shared between threads

// Variables (i, j, k) are private per-thread

#pragma omp parallel for shared(a, b, result) private

(i, j, k) for (i = 0; i &lt; size; i++) for (j = 0; j &lt; size; j++) for (k = 0; k &lt; size; k++) result[i][j] += a[i][k] * b[k][j];

}

OpenMP is a set of compiler directives and library routine directly supported by GCC (GNU C-Compiler) to specify high level parallelism in C/C++ or Fortran programs. In this example, we use OpenMP to create multiple threads, each working on one iteration of the outer-most loop.

The line beginning with #pragma directs the compiler to generate the code that parallelizes the for loop. The compiler will split the for loop iterations into different chunks (each chunk contains one or more iterations) which will be executed on different OpenMP threads in parallel.

• Compile the code in a terminal (console):

&gt; gcc -fopenmp -o mm1 mm-omp.c

<ul>

 <li>The –fopenmp flag enables the compiler to detect the #pragma commands, which would otherwise be ignored by the compiler.</li>

 <li>To execute the OpenMP program, you run it as a normal program. To multiply two square matrices of size 10, using 4 threads.</li>

 <li>Run the program in a terminal (console):</li>

</ul>

&gt; ./mm1 10 4

<table width="601">

 <tbody>

  <tr>

   <td width="70"></td>

   <td width="531"><strong><u>Exercise 2</u></strong>Compile and run the matrix multiplication program. Modify the number of threads and observe the trend in execution time. You may want to use a relatively large matrix to really stress the processor cores.</td>

  </tr>

 </tbody>

</table>

<h1>Part 2: Performance Instrumentation</h1>

<h2>Processor Hardware Event Counters</h2>

Due to the multiple layers of abstraction in modern high level programming, it is sometime hard to understand performance at the hardware level. For example, a single line of code <strong>result[i][j] += a[i][k] * b[k][j]; </strong>could be translated into a handful of machine instructions. In addition, this statement can take a wide range of execution time to finish depending on cache / memory behavior.

Hardware event counters are special registers built into modern processors that can be used to count low-level events in a system such as the number of instructions executed by a program, number of L1 cache misses among others. A modern processor such as Core i5 or Core i7 supports a few hundred types of events.

In this section, we will learn how to get hardware events counters for measuring the performance of a program using <strong>perf</strong>, a Linux OS utility. <strong>perf </strong>enables profiling of the entire execution of a program and produces a summary profile as output.

<ul>

 <li>Use <strong>perf stat </strong>to produce a summary of program performance</li>

</ul>

 &gt; perf stat — ./mm0

<ul>

 <li>perf output shown below. (program output not shown)</li>

</ul>

Performance counter stats for ‘./mm0’:

45.595495 task-clock                                                  #                   0.657 CPUs utilized

410 context-switches                                  #              0.009 M/sec

0 CPU-migrations                                      #              0.000 M/sec

362 page-faults                                              #              0.008 M/sec

98,015,090 cycles                                                            #              2.150 GHz

35,472,062 stalled-cycles-frontend                                                                        #                   36.19% frontend cycles idle

10,198,393 stalled-cycles-backend               #              10.40% backend cycles idle 169,428,906 instructions #              1.73 insns per cycle

#                                       0.21 stalled cycles per insn

21,957,507 branches                                                            # 481.572 M/sec

167,305 branch-misses                                          #                   0.76% of all branches

0.069367093 seconds time elapsed

<ul>

 <li>To count the events of interest, you can specify exactly which events you wish to measure:</li>

</ul>

&gt; perf stat -e cache-references -e cache-misses e cycles -e instructions — ./mm.0

<ul>

 <li>perf output shown below. (program output not shown)</li>

</ul>

Performance counter stats for ‘./mm.0 ‘:

8,764,236 cache-references

58,695 cache-misses                                            #                      0.670 % of all cache refs

5,766,321,978 cycles                                                                                                 #              0.000 GHz

10,542,104,914 instructions                                                #                     1.83 insns per cycle

2.151194898 seconds time elapsed

<ul>

 <li>To list all the events available under your platform, you can use the command:</li>

</ul>

&gt; perf list

<ul>

 <li>perf output shown below.</li>

</ul>

List of pre-defined events (to be used in -e):

cpu-cycles OR cycles          [Hardware event] stalled-cycles-frontend OR idle-cycles-frontend          [Hardware event]

stalled-cycles-backend OR idle-cycles-backend    [Hardware event] instructions          [Hardware event]

cache-references                                                                                                        [Hardware event]…

[ perf list output is truncated …]

<table width="601">

 <tbody>

  <tr>

   <td width="69"></td>

   <td width="532"><strong><u>Exercise 3</u></strong>Use perf to profile the OpenMP version of matrix multiplication using different number of threads. Observe the variation of different performance event counts for different runs.</td>

  </tr>

 </tbody>

</table>

<h2>Processor Hardware Event Counters</h2>

In this section, we will learn how to use hardware event counters to analyze the performance of a parallel program. For the exercises below, run the OpenMP version of matrix multiplication code on the both the Desktop PC (Intel Core i7) and the Server Tower (Intel Xeon) with n = 1, 2, 4, 8, 16, 32 threads. You should choose a reasonable matrix size (or have a few different test scenarios).

<table width="601">

 <tbody>

  <tr>

   <td width="60"></td>

   <td width="541"><strong><u>Exercise 4</u></strong>Determine (i) the number of instruction executed per cycles (IPC) and (ii) MFLOPS Comment on how IPC and MFLOPS change with increasing number of threads. [Hint: To get the MFLOPS, you need to estimate how many floating point operations are executed during program execution]</td>

  </tr>

  <tr>

   <td width="60"></td>

   <td width="541"><strong><u>Exercise 5</u></strong>In mm-omp.c, when the multiplication is performed, we read the matrices and multiply in row-major order. Modify mm-omp.c (new name mm-omp-col.c) to perform the multiplication in column major order. Read more about row major and colum major ordering: <a href="https://en.wikipedia.org/wiki/Row-_and_column-major_order">https: </a><a href="https://en.wikipedia.org/wiki/Row-_and_column-major_order">//en.wikipedia.org/wiki/Row-_and_column-major_order</a></td>

  </tr>

  <tr>

   <td width="60"></td>

   <td width="541"><strong><u>Exercise 6</u></strong>Compile and run mm-omp.c and the mm-omp-col.c for different number of threads and record execution time for both versions. Comment on the observations.</td>

  </tr>

 </tbody>

</table>

<sup></sup><strong>Homework:</strong>You are required to produce a write-up with the results for exercises 4 to 6. Submit the lab report

(in PDF form with file name format A0123456X.pdf) via LumiNUS. The document must contain explanation on how you have solved the exercises, the results and the raw measured data.

<h1>Appendix: OpenMP Programming</h1>

<h2>Structure of an OpenMP Program</h2>

An OpenMP program consists of several parallel regions interleaved with sequential sections. In each parallel block, there is always one master thread and there may be several slave threads. The master thread always has thread id of zero.

Below (hello-omp.c) is the OpenMP version of canonical hello world program. Each parallel region starts with a pragma command that tells the compiler that the following code block will be executed in parallel. #include &lt;omp.h&gt;

#include &lt;stdio.h&gt; #include &lt;stdlib.h&gt;

int main (int argc, char *argv[])

{ int thread_id, no_threads;

/* Fork slave threads, each with its own unique thread id */

#pragma omp parallel

{

/* Obtain thread id */

thread_id = omp_get_thread_num();

printf(“Hello World from thread = %d
”, thread_id);

/* Only master thread does this. Master thread always has id equal to 0 */

if (thread_id == 0)

{ no_threads = omp_get_num_threads();

printf(“Number of threads = %d
”, no_threads);

}

} /* All slave threads join master thread and are destroyed */

}

If you compile and run this program on the Intel Core i7 machine, you will see that there are 8 threads that echo the “Hello World” string. By default, OpenMP creates a number of threads equal to the number of processor cores of the machine. You can change this using the function omp_set_num_threads(int) in your OpenMP code, or the environment variable OMP_NUM_THREADS.

<h2>Work-sharing Constructs</h2>

Inside the parallel region, there is some work that needs to be done. OpenMP provides four ways in which the work can be partitioned among the thread. These constructs are called work-sharing constructs:

<ul>

 <li>Loop Iterations: Iterations within a for loop will be split among the existing threads. The programmer can control the order and the number of iterations assigned to each thread using the schedule directive. Example:</li>

</ul>

#pragma omp parallel

{

#pragma omp for schedule (static, chunksize)

for (i = 0; i &lt; n; i++)

x[i] = y[i];

}

In this example, n iterations of the for loop is divided into pieces of size, chunksize, and assigned statically to the threads. There are other options for schedule, which you can read in the OpenMP quick-reference.

<ul>

 <li>Sections: The programmer manually defines some code blocks that will be assigned to any available thread, one at a time. Example:</li>

</ul>

#pragma omp parallel

{

#pragma omp sections

{

#pragma omp section

{ work1();

}

#pragma omp section

{

work2();

}

#pragma omp section

{ work3();

}

}

}

In the sections region, you see the declaration of three work sections. The sections may be passed to different threads for execution.

<ul>

 <li>Single section: Only a single thread will execute the code. The runtime decides which thread will get to execute.</li>

 <li>Master section: Similar to single section, only that the master thread executes the code.</li>

</ul>

<h2>Synchronization Constructs</h2>

OpenMP provides multiple directives to manage critical sections (we learned there with phtreads in Lab01) in code.

<ul>

 <li>master directive: Specifies a region that must be executed only by the master thread.</li>

</ul>

#pragma omp master

structured_block

<ul>

 <li>critical directive: Specifies a critical region that must be executed only by one thread at a time.</li>

</ul>

(example below)

#include &lt;omp.h&gt; main(int argc, char *argv[]) {

int x; x = 0;

#pragma omp parallel shared(x)

{

#pragma omp critical

x = x + 1;

} /* end of parallel region */

}

<ul>

 <li>atomic directive: Works like a mini-critical section; specified that a specific memory location must be updated atomically.</li>

</ul>

#pragma omp atomic

<sup></sup>statement_expression

<ul>

 <li>barrier directive: synchronizes all thread (threads wait until all thread are completed.</li>

</ul>

#pragma omp barrier <strong><u>Resources</u></strong>

<ul>

 <li>For more details on OpenMP constructs, please refer to LLNL OpenMP documentation at <a href="https://computing.llnl.gov/tutorials/openMP/">https://computing.llnl.gov/tutorials/openMP/</a></li>

 <li>OpenMP supports Microsoft Visual C/C++ compiler too. Most of the examples you find there should work on Linux with gcc or g++ compiler as well. You can learn more at <a href="https://msdn.microsoft.com/en-us/library/tt15eb9t.aspx">https://msdn.microsoft.com/en-us/library/tt15eb9t.aspx</a></li>

 <li>perf reference <a href="https://perf.wiki.kernel.org/index.php/Main_Page">https://perf.wiki.kernel.org/index.php/Main_Page</a></li>

 <li>perf manual: &gt; man perf</li>

 <li>Performance Analysis Guide for Intel Core i7 Processor and Intel Xeon 5500 processor <a href="https://software.intel.com/sites/products/collateral/hpc/vtune/performance_analysis_guide.pdf">http://software.intel.com/sites/products/collateral/hpc/vtune/ </a><a href="https://software.intel.com/sites/products/collateral/hpc/vtune/performance_analysis_guide.pdf">pdf</a></li>

</ul>

OpenMP Reference Sheet for C/C++ <a href="http://www.plutospin.com/files/OpenMP_reference.pdf">http://www.plutospin.com/files/OpenMP_reference.pdf</a>