Download Link: https://assignmentchef.com/product/solved-cs100-lab-04-valgrind
<br>
<h1>Memory management</h1>

The ability to dynamically allocate and deallocate memory is one of the strongest features of C++ programming, but the greatest strength can also be the greatest weakness. This is certainly true of C++ applications, where memory-handling problems are among the most common bugs.

One of the most subtle and hard-to-detect bugs is the memory leak – the failure to properly deallocate memory that was previously allocated. A small memory leak that occurs only once may not be noticed, but programs that leak large amounts of memory, or leak progressively, may display symptoms ranging from poor (and gradually decreasing) performance to running out of memory completely. Worse, a leaking program may use up so much memory that it causes another program to fail, leaving the user with no clue to where the problem truly lies. In addition, even harmless memory leaks may be symptomatic of other problems.

<h2><a id="user-content-valgrind" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#valgrind" aria-hidden="true"></a>Valgrind</h2>

<a href="https://valgrind.org/" rel="nofollow">Valgrind</a> is an instrumentation framework for building dynamic analysis tools. We will be focusing on the Memcheck tool in their extensive tool suite.

<blockquote>

 Note: Valgrind is designed for Linux, but it is also compatible with some versions of Mac OS (unless you are running a very new version of the OS). If you are performing this lab on Windows, it is recommended that you use <code>hammer</code> to run valgrind.

</blockquote>

<h3><a id="user-content-memcheck" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#memcheck" aria-hidden="true"></a>Memcheck</h3>

<strong>Memcheck</strong> detects memory-management problems, and is aimed primarily at C and C++ programs. When a program is run under Memcheck’s supervision, all reads and writes of memory are checked, and calls to <code>malloc</code>/<code>new</code>/<code>free</code>/<code>delete</code> are <em>intercepted</em>. As a result, Memcheck can detect if your program:

<ul>

 <li>Accesses memory it shouldn’t (areas not yet allocated, areas that have been freed, areas past the end of heap blocks, inaccessible areas of the stack).</li>

 <li>Uses uninitialized values in dangerous ways.</li>

 <li>Leaks memory.</li>

 <li>Does bad frees of heap blocks (double frees, mismatched frees).</li>

 <li>Passes overlapping source and destination memory blocks to <code>memcpy()</code> and related functions.</li>

</ul>

Memcheck reports these errors as soon as they occur, giving the source line number at which it occurred, and also a stack trace of the functions called to reach that line. Memcheck tracks valid addresses and initialization of values. As a result, it can detect the use of uninitialized memory. Memcheck runs programs about 10-30x slower than normal.

<h3><a id="user-content-perparing-your-program" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#perparing-your-program" aria-hidden="true"></a>Perparing your program</h3>

Compile your program with the following flags:

<ul>

 <li><code>-g</code> – to include debugging information so that Memcheck’s error messages include <em>exact</em> line numbers.</li>

 <li><code>-O0</code> – to turn off all optimizations. Memcheck’s error messages can be slightly inaccurate at <code>-O1</code> and can lead to spurious errors at <code>-O2</code> or above.</li>

</ul>

For this lab the command:

will be sufficient.

<h3><a id="user-content-running-your-program-under-memcheck" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#running-your-program-under-memcheck" aria-hidden="true"></a>Running your program under Memcheck</h3>

The examples/exercises in this lab don’t use command line arguments, but if your program is called like this:

Then run Valgrind like this:

Memcheck is the default tool so no tool flags are necessary. The <code>--leak-check</code> option turns on the detailed memory leak detector.

Your program will run much slower (e.g. 20 – 30 times) than normal and use a lot more memory. Memcheck will issue messages about memory errors and leaks that it detects.

<h2><a id="user-content-examples" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#examples" aria-hidden="true"></a>Examples</h2>

Let’s go over some simple examples of the types of errors you can uncover using Valgrind and how to interpret the error messages. We’re going to start with a simple c++ program (save the file as simple.cpp):

This is a pretty typical introduction to pointers example from CS012. It also contains a <strong>very</strong> common memory management mistake. On line 2 we allocated a contiguous location in memory large enough for 10 <code>int</code>s and assigned the starting address to an <code>int</code> pointer <code>p</code>. The next line then attempts to assign to the element with index 10. This may seem fine since it <em>seems</em> to be the last element in the array; however, due to 0-based addressing, the last element is actual <code>p[9]</code>.

Now, let’s compile the program:

And run it:

You may get results similar to:

Or it might even run, either way figuring out your error is cryptic and difficult. Let’s run it through Valgrind (you don’t need to recompile):

And you’ll get a much more in detailed report of what is going on:

Let’s look at this bit by bit.

<ul>

 <li>13621 is the process ID; typically it is not important.</li>

 <li>The rest shows some copyright and version information as well as the command Valgrind is running (your executable).</li>

</ul>

Now we get into the errors:

This shows us that the error was an <code>Invalid write of size 4</code> on line <code>(simple.cpp:3)</code> in function <code>main</code>. The address of the line (<code>0x108698</code>) as well as the address of the memory (<code>0x5b7dca8</code>) are shown here and they can be incredibly useful when debugging some really nasty memory bugs on the stack/heap, but they aren’t helpful in this simple case. The rest of the error message will provide additional information that may help in identifying and fixing the error, in this case, a part of the root cause of the error comes from line (<code>simple.cpp:2</code>) with the <code>operator new[](unsigned long)</code>.

After this we get a <code>HEAP SUMMARY</code>:

This shows us the second error we have. We can see here that we have <code>40 bytes in 1 blocks</code> still <em>in use at exit</em>. Additionally, we had <code>2 allocs</code> but only <code>1 free</code>. There is memmory that was allocated but then did not have a corresponding deallocation. Looking at the records (second half of the message), we can see that <code>40 bytes in 1 block</code> are <strong>definitely lost</strong> in loss record 1 of 1. We are given some additional information to help track it down. <code>operator new[](unsigned long)</code> at <code>main (simple.cpp:2)</code> is at fault here.

Finally, we are given a summary:

We have <code>40 bytes in 1 blocks</code> <strong>definitely lost</strong> and nothing <em>indirectly</em>, or <em>possibly</em> lost, no memory <em>still reachable</em> and no errors were suppressed. Don’t worry, the next examples will cover these types of errors as well. You can add to verbosity by adding additional <code>-v</code> flags, but the default typically has enough information to cover most errors.

Let’s fix the errors one at a time. First, let’s get rid of the <code>Invalid write</code> since that is causing our program to crash. The fix is quite simple, figure out the size of the array, the last index in the array, and make sure you are in bounds of the array.

And compile and run again (I add a trick I frequently use, the <code>&amp;&amp;</code> means “execute the second command <strong>only if</strong> the first command <strong>succeeds</strong>):

Now your program should run to completion (you won’t see any output). Woohoo! You fixed it…wait, there were two errors weren’t there? Let’s run it back through Valgrind and see:

And you’ll see the following (tool information excluded for brevity):

We can clearly see from our summaries that we are still <em>leaking</em> memory even though we don’t have anymore write errors. We have the same error as above, but to remind us, we have <code>40 bytes in 1 blocks</code> <strong>definitely lost</strong> in record 1 due to the <code>operator new[](unsigned long)</code> on line (<code>simple.cpp:2</code>) in <code>main</code> function. Let’s look at that code block (between <code>{</code> and <code>}</code>):

The <code>new</code> operator allocates new memory, but we never deallocate that memory. Let’s do that now:

<em>Yes I used the wrong delete but this was done on purpose, bear with me. If you didn’t realize that was the wrong delete, that’s exactly what this tool helps with so don’t feel bad!</em>

<h3><a id="user-content-when-a-block-is-freed-with-an-inappropriate-deallocation-function" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#when-a-block-is-freed-with-an-inappropriate-deallocation-function" aria-hidden="true"></a>When a block is freed with an inappropriate deallocation function</h3>

<ul>

 <li>If allocated with <code>malloc</code>, <code>calloc</code>, <code>realloc</code>, <code>valloc</code>, or <code>memalign</code>, you must deallocate with free*.</li>

 <li>If allocated with <code>new[]</code>, you must deallocate with <code>delete[]</code>.</li>

 <li>If allocated with <code>new</code>, you must deallocate with <code>delete</code>.</li>

</ul>

* These are common in <code>C</code> but less commonly seen in <code>C++</code>, so you likely haven’t seen them much at this point.

Let’s run this through Valgrind again:

If we look at the summary it appears we freed all of the memory, however, if we look at the error list we will see that we used a <code>Mismatched free() / delete / delete []</code> at <code>operator delete(void*)</code> on line (<code>simple.cpp:5</code>) in the <code>main</code> function. And if we look at the additional information we can see the root is from <code>operator new[](unsigned long)</code> at line (<code>simple.cpp:2</code>) in the <code>main</code> function. Looking at these two operators as shown by Valgrind, we can see that we allocated with a <code>[]</code> but didn’t deallocate with the same. Let’s fix that now.

And run Valgrind again:

Now we’re at what is called <strong>Memcheck-clean</strong>, we have <code>0 bytes in 0 blocks</code> in use at exit, <code>All heap blocks were freed -- no leaks are possible</code> and we have <code>0 errors from 0 contexts</code>. This is the goal of any program we write.

The final step, now that we’re at <strong>Memcheck-clean</strong>, is to re-run with optimizations turned back on:

Congratulations! You’ve successfully cleaned up a simple program from all memory leaks using the Memcheck tool in the Valgrind suite.

<h3><a id="user-content-illegal-readillegal-write-errors" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#illegal-readillegal-write-errors" aria-hidden="true"></a>Illegal read/illegal write errors</h3>

The first example showed an example of an <em>Illegal write</em> error. If we had tried to print out the value

We would have had a reported <code>Invalid read of size 4</code> at that line. The rest of that error message looks very similar to the <code>Invalide write</code> error.

<h3><a id="user-content-use-of-uninitialized-values" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#use-of-uninitialized-values" aria-hidden="true"></a>Use of uninitialized values</h3>

As a general rule, you should always initialize your values. Nonetheless there are times that it happens (unintentional or not). Memcheck will catch these errors with a caveat. Let’s look at the following program (<code>uninitialized.cpp</code>):

Yes, it is again a silly examply, but it is used for illustrative purposes so bear with me. Let’s compile and run this program:

This should run and will print a number (often 0), <strong>but</strong> I want to emphasize that this should not be expected and is not because of the C++ standard. <code>x</code> is unintialized, and you may very well get a junk value. (I have graded a lot of assignments where students submitted programs that worked for them because their uninitialized variables happened to be zero. When I graded them on my computer, they held junk values, and their programs failed. The only way to be sure is to test your code under valgrind.) Now, let’s run it through Valgrind and see what happens:

Oh boy…that’s a lot of errors, and they really don’t seem friendly to look at. This is similar to compiler errors generated when using the Standard Template Library (STL), <code>cout</code> in this case. Let’s take a look at the first error though:

Okay…so there is a <em>Conditional jump or move</em> somewhere that depends on an <em>unitialised value(s)</em>. You may have learned about <em>jumps</em> in CS061 if you’ve taken that course so far but the important part here is that it <em>depends on an unitialised value(s)</em>. In fact, if we look at the second error we can see:

There it is, right at the top: <code>Use of unintialised value of size 8</code>. However, if we inspect the error message we see that it is at <code>???</code> by <code>std::ostreambuf...</code> by <code>std::ostream&amp;...</code> by <code>main (uninitialized.cpp:6)</code>. That last <em>by…by…by</em> sequence is the stack trace. It’s frequently a good idea to skip past all the library files and see what is the last function <strong>you wrote</strong> that caused that. We can see that in the <code>main</code> function on line 6 we set off this error. But we still don’t know what caused it. We can see by inspecting line 6 <code>cout &lt;&lt; x &lt;&lt; endl;</code> that it is likely caused by the variable <code>x</code>, but that’s about all we get. It turns out, we can add additional flags into the Valgrind call to get more out of this, specifically the <code>--track-origins=yes</code> flag:

Now let’s look at that same error again (the second one):

I’ve truncated the errors from the STL for brevity. You shouldn’t need to dig into those in most errors. If you do, grab a cup of tea, clear your schedule for the week and get busy. Now we can see that the new flag added some additional information:

The variable at fault here was created by stack allocation in the <code>main</code> function from (<code>unitialized.cpp:4</code>). After some inspection of that function we can see that the variable <code>x</code> is at fault and can fix it by simply initializing it to some value, let’s say 10.

Now, compiling and running Valgrind again we can see that we achieved <strong>Memcheck-clean</strong>.

It is important to note here that Valgrind will let your program copy around junk data as much as it likes. Memcheck will observe and keep track of the data, but doesn’t complain. It doesn’t complain until the use of uninitialized data might affect your program’s externally-visible behaviour. Experiment with the following program to see how the generated reports look:

Another interesting example is this one:

When I run this example, the valgrind report flags line 5 (the std::cout line), line 14 (the call to foo) in the stack trace. It points to line 9 (the open brace on the main function) as the origin. The origin is actually line 10, where the x is declared. Note that the test for uninitialized values occurs when the value is used in a conditional, not on arithematic. Thus, valgrind will not flag the computation of y. A handy debugging trick is to insert “dummy” conditions to trigger the test, allowing you to test variables one by one to track down the problem.

<h3><a id="user-content-illegal-frees" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#illegal-frees" aria-hidden="true"></a>Illegal frees</h3>

Memcheck will also track the memory that has been deallocated so if you try to re-deallocate memory (as in a double free) it will catch that and report it to you. Consider the following program (doubleFree.cpp):

By inspection it is easy to see that <code>p</code> has been <code>delete</code>d twice, but let’s run it through Valgrind anyways:

The first (and only) error shows us that we have an <code>Invalid free() / delete / delete[] / realloc()</code> at <code>operator delete(void*)</code> on line (<code>doubleFree.cpp:5</code>) in the <code>main</code> function. This memory was <code>free'd</code> by <code>operator delete(void*)</code> on line (<code>doubleFree.cpp:4</code>) in the <code>main</code> function. The block was also originally <code>alloc'd</code> at <code>operator new(unsigned long)</code> on line <code>doubleFree.cpp:2</code> in the <code>main</code> function. We can get to <strong>Memcheck-clean</strong> by simply removing either of the <code>delete p;</code> statements.

<h3><a id="user-content-memory-leak-detection" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#memory-leak-detection" aria-hidden="true"></a>Memory leak detection</h3>

Memcheck has the following four leak kinds:

<ul>

 <li>“Still reachable” – A start-pointer or chain of start-pointers to the block is found. The program could have <em>in theory</em> deallocated the memory.</li>

 <li>“Definitely lost” – No pointer to the block can be found. There is no possible way to have deallocated this memory before the program exited.</li>

 <li>“Indirectly lost” – The block is lost, not because there are no pointers to it, but rather because all the blocks that point to it are themselves lost. For example, if you have a binary tree and the root node is lost, all it’s children are indirectly lost. If you fixed the “definitely lost” block correctly these will be fixed as a side effect.</li>

 <li>“Possibly lost” – A chain of one or more pointers to the block have been found, but at least one of them is an <em>interior</em> pointer. This is typically not good unless, through inspection, you identify the interior pointer and now how to access this memory.</li>

</ul>

A <strong>start-pointer</strong> is a pointer at the beginning of a block whereas an <strong>interior-pointer</strong> points somewhere in the middle of the block (intentionally or unintentionally), for example:

<ul>

 <li><code>p</code> points to the beginning of the array and is a start-pointer</li>

 <li><code>ip</code> points to the middle of the array and is an interior-pointer</li>

</ul>

There are many more kinds of interior pointers and you can read more about them on your own.

<h2><a id="user-content-lab-exercise" class="anchor" href="https://github.com/FireFly0000/lab04-valgrind#lab-exercise" aria-hidden="true"></a>Lab exercise</h2>

Now use what you have learned from this lab to get the <code>lineage</code> program to <strong>Memheck-clean</strong>. The program is a simplified family tree manager. It maintains a list of <code>Person</code>s in a <code>PersonList</code>. Each <code>Person</code> object maintains a set of pointers to his/her parents as well as to his/her children. The code contains several memory leaks. To compile:

* The -fno-inline instructs the compiler to not inline any functions and makes it easier to see the function call chain.

Some things to consider:

<ul>

 <li>As with compiler errors, it’s a good idea to start at the top and see if fixing those errors fix later errors.</li>

 <li>Sometimes by fixing an error you <em>introduce</em> more. These aren’t new errors, they were just revealed by fixing an earlier error.</li>

 <li>Think carefully about what <code>delete</code> vs. <code>delete[]</code> do and why we spent time in CS014 discussing <em>shallow</em> vs. <em>deep</em> copies.</li>

 <li>If you get to only (or mostly) “indirectly lost” errors you’ll notice that Valgrind doesn’t report each one individually. This is because they aren’t “lost” so much as “forgotten” by the programmer. If this happens you can turn on <code>--show-reachable=yes</code> to show more details about each of those errors.</li>

</ul>

5/5 - (1 vote)

<pre><code>$ g++ -g -O0 *.cpp -o &lt;exec_name&gt;</code></pre>

<pre><code>$ ./myprog arg1 arg2</code></pre>

<pre><code>$ valgrind --leak-check=full ./myprog arg1 arg2</code></pre>

<pre><code>int main() {    int *p = new int[10];    p[10] = 1;    return 0;}</code></pre>

<pre><code>$ g++ -g -O0 -o example1 simple.cpp</code></pre>

<pre><code>$ ./example1</code></pre>

<pre><code>example1: malloc.c:2401: sysmalloc: Assertion `(old_top == initial_top (av) &amp;&amp; old_size == 0) || ((unsigned long) (old_size) &gt;= MINSIZE &amp;&amp; prev_inuse (old_top) &amp;&amp; ((unsigned long) old_end &amp; (pagesize - 1)) == 0)' failed.Aborted (core dumped)</code></pre>

<pre><code>$ valgrind --leak-check=full ./example1</code></pre>

<pre><code>==13621== Memcheck, a memory error detector==13621== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.==13621== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info==13621== Command: ./example1==13621== ==13621== Invalid write of size 4==13621==    at 0x108698: main (simple.cpp:3)==13621==  Address 0x5b7dca8 is 0 bytes after a block of size 40 alloc'd==13621==    at 0x4C3089F: operator new[](unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==13621==    by 0x10868B: main (simple.cpp:2)==13621== ==13621== ==13621== HEAP SUMMARY:==13621==     in use at exit: 40 bytes in 1 blocks==13621==   total heap usage: 2 allocs, 1 frees, 72,744 bytes allocated==13621== ==13621== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1==13621==    at 0x4C3089F: operator new[](unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==13621==    by 0x10868B: main (simple.cpp:2)==13621== ==13621== LEAK SUMMARY:==13621==    definitely lost: 40 bytes in 1 blocks==13621==    indirectly lost: 0 bytes in 0 blocks==13621==      possibly lost: 0 bytes in 0 blocks==13621==    still reachable: 0 bytes in 0 blocks==13621==         suppressed: 0 bytes in 0 blocks==13621== ==13621== For counts of detected and suppressed errors, rerun with: -v==13621== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)</code></pre>

<pre><code>==13621== Memcheck, a memory error detector==13621== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.==13621== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info==13621== Command: ./example1</code></pre>

<pre><code>==13621== Invalid write of size 4==13621==    at 0x108698: main (simple.cpp:3)==13621==  Address 0x5b7dca8 is 0 bytes after a block of size 40 alloc'd==13621==    at 0x4C3089F: operator new[](unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==13621==    by 0x10868B: main (simple.cpp:2)</code></pre>

<pre><code>==13621== HEAP SUMMARY:==13621==     in use at exit: 40 bytes in 1 blocks==13621==   total heap usage: 2 allocs, 1 frees, 72,744 bytes allocated==13621== ==13621== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1==13621==    at 0x4C3089F: operator new[](unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==13621==    by 0x10868B: main (simple.cpp:2)</code></pre>

<pre><code>==13621== LEAK SUMMARY:==13621==    definitely lost: 40 bytes in 1 blocks==13621==    indirectly lost: 0 bytes in 0 blocks==13621==      possibly lost: 0 bytes in 0 blocks==13621==    still reachable: 0 bytes in 0 blocks==13621==         suppressed: 0 bytes in 0 blocks==13621== ==13621== For counts of detected and suppressed errors, rerun with: -v==13621== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)</code></pre>

<pre><code>int main() {    int *p = new int[10];    p[9] = 1; // Correct this to size - 1 = 10 - 1 = 9    return 0;}</code></pre>

<pre><code>$ g++ -g -O0 simple.cpp -o example1 &amp;&amp; ./example1</code></pre>

<pre><code>$ valgrind ./example1</code></pre>

<pre><code>==14490== HEAP SUMMARY:==14490==     in use at exit: 40 bytes in 1 blocks==14490==   total heap usage: 2 allocs, 1 frees, 72,744 bytes allocated==14490== ==14490== 40 bytes in 1 blocks are definitely lost in loss record 1 of 1==14490==    at 0x4C3089F: operator new[](unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==14490==    by 0x10868B: main (simple.cpp:2)==14490== ==14490== LEAK SUMMARY:==14490==    definitely lost: 40 bytes in 1 blocks==14490==    indirectly lost: 0 bytes in 0 blocks==14490==      possibly lost: 0 bytes in 0 blocks==14490==    still reachable: 0 bytes in 0 blocks==14490==         suppressed: 0 bytes in 0 blocks==14490== ==14490== For counts of detected and suppressed errors, rerun with: -v==14490== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)</code></pre>

<pre><code>{    int *p = new int[10];    p[9] = 1;    return 0;}</code></pre>

<pre><code>{    int *p = new int[10];    p[9] = 1;    delete p; // Deallocate the memory from the new above    return 0;}</code></pre>

<pre><code>==14593== Mismatched free() / delete / delete []==14593==    at 0x4C3123B: operator delete(void*) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==14593==    by 0x10871E: main (simple.cpp:5)==14593==  Address 0x5b7dc80 is 0 bytes inside a block of size 40 alloc'd==14593==    at 0x4C3089F: operator new[](unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==14593==    by 0x1086FB: main (simple.cpp:2)==14593== ==14593== ==14593== HEAP SUMMARY:==14593==     in use at exit: 0 bytes in 0 blocks==14593==   total heap usage: 2 allocs, 2 frees, 72,744 bytes allocated==14593== ==14593== All heap blocks were freed -- no leaks are possible==14593== ==14593== For counts of detected and suppressed errors, rerun with: -v==14593== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)</code></pre>

<pre><code>{    int *p = new int[10];    p[9] = 1;    delete[] p; // Deallocate the memory from the new above    return 0;}</code></pre>

<pre><code>==14620== HEAP SUMMARY:==14620==     in use at exit: 0 bytes in 0 blocks==14620==   total heap usage: 2 allocs, 2 frees, 72,744 bytes allocated==14620== ==14620== All heap blocks were freed -- no leaks are possible==14620== ==14620== For counts of detected and suppressed errors, rerun with: -v==14620== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)</code></pre>

<pre><code>$ g++ -g -O2 *.cpp -o example1</code></pre>

<pre><code>cout &lt;&lt; p[10] &lt;&lt; endl;</code></pre>

<pre><code>#include &lt;iostream&gt;using namespace std;int main() {    int x;    cout &lt;&lt; x &lt;&lt; endl;    return 0;}</code></pre>

<pre><code>$ g++ -g -O0 uninitialized.cpp -o example2 &amp;&amp; ./example2</code></pre>

<pre><code>==14854== Conditional jump or move depends on uninitialised value(s)==14854==    at 0x4F43B2A: std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; std::num_put&lt;char, std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; &gt;::_M_insert_int&lt;long&gt;(std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt;, std::ios_base&amp;, char, long) const (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F50074: std::ostream&amp; std::ostream::_M_insert&lt;long&gt;(long) (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x1088A2: main (uninitialized.cpp:6)==14854== ==14854== Use of uninitialised value of size 8==14854==    at 0x4F4362E: ??? (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F43B53: std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; std::num_put&lt;char, std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; &gt;::_M_insert_int&lt;long&gt;(std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt;, std::ios_base&amp;, char, long) const (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F50074: std::ostream&amp; std::ostream::_M_insert&lt;long&gt;(long) (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x1088A2: main (uninitialized.cpp:6)==14854== ==14854== Conditional jump or move depends on uninitialised value(s)==14854==    at 0x4F4363B: ??? (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F43B53: std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; std::num_put&lt;char, std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; &gt;::_M_insert_int&lt;long&gt;(std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt;, std::ios_base&amp;, char, long) const (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F50074: std::ostream&amp; std::ostream::_M_insert&lt;long&gt;(long) (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x1088A2: main (uninitialized.cpp:6)==14854== ==14854== Conditional jump or move depends on uninitialised value(s)==14854==    at 0x4F43B86: std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; std::num_put&lt;char, std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; &gt;::_M_insert_int&lt;long&gt;(std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt;, std::ios_base&amp;, char, long) const (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F50074: std::ostream&amp; std::ostream::_M_insert&lt;long&gt;(long) (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x1088A2: main (uninitialized.cpp:6)==14854== 0==14854== ==14854== HEAP SUMMARY:==14854==     in use at exit: 0 bytes in 0 blocks==14854==   total heap usage: 2 allocs, 2 frees, 73,728 bytes allocated==14854== ==14854== All heap blocks were freed -- no leaks are possible==14854== ==14854== For counts of detected and suppressed errors, rerun with: -v==14854== Use --track-origins=yes to see where uninitialised values come from==14854== ERROR SUMMARY: 4 errors from 4 contexts (suppressed: 0 from 0)</code></pre>

<pre><code>==14854== Conditional jump or move depends on uninitialised value(s)==14854==    at 0x4F43B2A: std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; std::num_put&lt;char, std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; &gt;::_M_insert_int&lt;long&gt;(std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt;, std::ios_base&amp;, char, long) const (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F50074: std::ostream&amp; std::ostream::_M_insert&lt;long&gt;(long) (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x1088A2: main (uninitialized.cpp:6)</code></pre>

<pre><code>==14854== Use of uninitialised value of size 8==14854==    at 0x4F4362E: ??? (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F43B53: std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; std::num_put&lt;char, std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt; &gt;::_M_insert_int&lt;long&gt;(std::ostreambuf_iterator&lt;char, std::char_traits&lt;char&gt; &gt;, std::ios_base&amp;, char, long) const (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x4F50074: std::ostream&amp; std::ostream::_M_insert&lt;long&gt;(long) (in /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.25)==14854==    by 0x1088A2: main (uninitialized.cpp:6)</code></pre>

<pre><code>$ valgrind --leak-check=full --track-origins=yes ./example2</code></pre>

<pre><code>==14904== Use of uninitialised value of size 8==14904==    ...==14904==    ...==14904==    ...==14904==    by 0x1088A2: main (uninitialized.cpp:6)==14904==  Uninitialised value was created by a stack allocation==14904==    at 0x10888A: main (uninitialized.cpp:4)</code></pre>

<pre><code>==14904==  Uninitialised value was created by a stack allocation==14904==    at 0x10888A: main (uninitialized.cpp:4)</code></pre>

<pre><code>int x = 10;</code></pre>

<pre><code>==14973== HEAP SUMMARY:==14973==     in use at exit: 0 bytes in 0 blocks==14973==   total heap usage: 2 allocs, 2 frees, 73,728 bytes allocated==14973== ==14973== All heap blocks were freed -- no leaks are possible==14973== ==14973== For counts of detected and suppressed errors, rerun with: -v==14973== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)</code></pre>

<pre><code>#include &lt;iostream&gt;using namespace std;int main() {    int x;    bool z;    int y = x + 5;    if (x) {        cout &lt;&lt; "X is non-zero" &lt;&lt; endl;    }    if (z) {        cout &lt;&lt; "Z is truthy" &lt;&lt; endl;    }    cout &lt;&lt; y &lt;&lt; endl;    return 0;}</code></pre>

<pre><code>#include &lt;iostream&gt;void foo(int x){    std::cout&lt;&lt;x&lt;&lt;std::endl;}int main(){    int x;    int r=4;    int y=x+r;    foo(y);        return 0;}</code></pre>

<pre><code>#include &lt;iostream&gt;void foo(int x){    std::cout&lt;&lt;x&lt;&lt;std::endl;}int zzzz=0; // dummy variableint main(){    int x;    int r=4;    int y=x+r;    if(x) zzzz++; // uninitialized error reported here    if(r) zzzz++;    if(y) zzzz++; // uninitialized error reported here    foo(y);        return 0;}</code></pre>

<pre><code>int main() {    int *p = new int(10);    delete p;    delete p;    return 0;}</code></pre>

<pre><code>==15125== Invalid free() / delete / delete[] / realloc()==15125==    at 0x4C3123B: operator delete(void*) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==15125==    by 0x108727: main (doubleFree.cpp:5)==15125==  Address 0x5b7dc80 is 0 bytes inside a block of size 4 free'd==15125==    at 0x4C3123B: operator delete(void*) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==15125==    by 0x108716: main (doubleFree.cpp:4)==15125==  Block was alloc'd at==15125==    at 0x4C3017F: operator new(unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)==15125==    by 0x1086FB: main (doubleFree.cpp:2)==15125== ==15125== ==15125== HEAP SUMMARY:==15125==     in use at exit: 0 bytes in 0 blocks==15125==   total heap usage: 2 allocs, 3 frees, 72,708 bytes allocated==15125== ==15125== All heap blocks were freed -- no leaks are possible==15125== ==15125== For counts of detected and suppressed errors, rerun with: -v==15125== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)</code></pre>

<pre><code>int *p = new int[10]int *ip = p + 2;</code></pre>