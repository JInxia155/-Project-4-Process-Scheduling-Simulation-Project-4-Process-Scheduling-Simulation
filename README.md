Download Link : https://programming.engineering/product/project-4-process-scheduling-simulation/

#  Project-4-Process-Scheduling-Simulation-Project-4-Process-Scheduling-Simulation
 🔍 Project 4: Process Scheduling Simulation Project 4: Process Scheduling Simulation


    Overview

In this project, you will implement a multiprocessor operating system simulator using a popular threading library for Linux called pthreads. The framework for the multiprocessor OS simulator is nearly complete, but missing one critical component: the process scheduler! Your task is to implement the process scheduler and three different scheduling algorithms.

The simulated operating system supports only one thread per process making it similar to the systems that we discussed in Chapter 6. However, the simulator itself will use a thread to represent each of the CPUs in the simulated hardware. This means that the CPUs in the simulator will appear to operate concurrently.

Note: Multiple CPU cores need to be enabled for this to work correctly. The CS 2200 Docker container should already be configured to run with 4 cores. Please see a TA if you are running into any problems.

We have provided you with source files that constitute the framework for your simulator. You will only need to modify answers.txt and student.c. However, just because you are only modifying two files doesn’t mean that you should ignore the other ones.

We have provided you the following files:

    os-sim.c – Code for the operating system simulator which calls your CPU scheduler.

    os-sim.h – Header file for the simulator.

    process.c – Descriptions of the simulated processes.

    process.h – Header file for the process data.

    student.c – This file contains stub functions for your CPU scheduler.

    student.h – Header file for your code to interface with the OS simulator. Also contains the ready queue struct definition.

    answers.txt – This is a text file that you should use to write your answers to the questions listed throughout the PDF.

Reminder: The only files that you need to edit arestudent.c and answers.txt. If you edit any other files, your code may fail the autograder!

1.1 Scheduling Algorithms

For your simulator, you will implement the following three CPU scheduling algorithms:

    First Come, First Serve (FCFS) – Runnable processes are kept in a ready queue. FCFS is non-preemptive; once a process begins running on a CPU, it will continue running until it either completes or blocks for I/O.

    Round-Robin – Similar to FCFS, except preemptive. Each process is assigned a timeslice when it is scheduled. At the end of the timeslice, if the process is still running, the process is preempted, and moved to the tail of the ready queue.

Project 4 CS 2200 – Systems and Networks Fall 2023

    Preemptive Priority Scheduling with Aging – Processes with higher priority get to run first and processes with lower priority get preempted for a process with higher priority. There is a caveat, though. Our priority scheduler factors in the age of a process when determining priority.

1.2 Process States

In our OS simulation, there are five possible states for a process. These states are listed in the process state t enum in os-sim.h:

    NEW – The process is being created, and has not yet begun executing.

    READY – The process is ready to execute, and is waiting to be scheduled on a CPU.

    RUNNING – The process is currently executing on a CPU.

    WAITING – The process has temporarily stopped executing, and is waiting for an I/O request to complete.

    TERMINATED – The process has completed.

There is a field named state in the PCB, which must be updated with the current state of the process. The simulator will use this field to collect statistics.

Figure 1: Process States

1.3 The Ready Queue

On most systems, there are a large number of processes that need to share the resources of a small number of CPUs. When there are more processes ready to execute than CPUs, processes must wait in the READY state until a CPU becomes available. To keep track of the processes waiting to execute, we keep a ready queue of the processes in the READY state.

Since the ready queue is accessed by multiple processors, which may add and remove processes from the ready queue, the ready queue must be protected by some form of synchronization. For this project, you will use a mutex lock that we have provided called ready mutex.

1.4 Scheduling Processes

Project 4 CS 2200 – Systems and Networks Fall 2023

Note that in a multiprocessor environment, we cannot mandate that the currently running process be at the head of the ready queue. There is an array (one entry for each CPU) that will hold the pointer to the PCB currently running on that CPU.

There is a special process, the “idle” process, which is scheduled whenever there are no processes in the READY state. This process simply waits for something new to be added to the ready queue and then calls schedule().

1.5 CPU Scheduler Invocation

There are five events which will cause the simulator to invoke schedule():

    yield() – A process completes its CPU operations and yields the processor to perform an I/O request.

    wake up() – A process that previously yielded completes its I/O request, and is ready to perform CPU operations. wake up() is also called when a process in the NEW state becomes runnable.

    preempt() – When using a Round-Robin scheduling algorithm, a CPU-bound process may be pre-empted before it completes its CPU operations.

    terminate() – A process exits or is killed.

    idle() – Waits for a new process to be added to the ready queue. idle() contains the code that gets executed by the idle process. In the real world, the idle process puts the processor in a low-power mode and waits. For our OS simulation, you will use a pthread condition variable to block the thread until a process enters the ready queue.

1.6 The Simulator

We will use pthreads to simulate an operating system on a multiprocessor computer. We will use one thread per CPU and one thread as a ‘supervisor’ for our simulation. The supervisor thread will spawn new processes (as if a user started a process). The CPU threads will simulate the currently-running processes on each CPU, and the supervisor thread will print output.

Since the code you write will be called from multiple threads, the CPU scheduler you write must be thread-safe! This means that all data structures you use, including your ready queue, must be protected using mutexes.

The number of CPUs is specified as a command-line parameter to the simulator. For this project, you will be performing experiments with 1, 2, and 4 CPU simulations.

Also, for demonstration purposes, the simulator executes much slower than a real system would. In the real world, a CPU burst might range from one to a few hundred milliseconds, whereas in this simulator, they range from 0.2 to 2.0 seconds.

Project 4 CS 2200 – Systems and Networks Fall 2023

Figure 2: Simulator Function Calls

The above diagram should give you a good overview of how the system works in terms of the functions being called and PCBs moving around.

Below is a second diagram that shows the entire system overview and the code that you need to write is inside of the green cloud at the bottom. All of the items outside of the green cloud are part of the simulator and will not need to be modified by you.

Project 4 CS 2200 – Systems and Networks Fall 2023

Figure 3: System overview

Project 4 CS 2200 – Systems and Networks Fall 2023

Compile and run the simulator with ./os-sim 2. After a few seconds, hit Control-C to exit. You will see the output below:

Figure 4: Sample Output

The simulator generates a Gantt Chart, showing the current state of the OS at every 100ms interval. The leftmost column shows the current time, in seconds. The next three columns show the number of Running, Ready, and Waiting processes, respectively. The next two columns show the process currently running on each CPU. The rightmost column shows the processes which are currently in the I/O queue, with the head of the queue on the left and the tail of the queue on the right.

As you can see, nothing is executing. This is because we have no CPU scheduler to select processes to execute! Once you complete Problem 1 and implement a basic FCFS scheduler, you will see the processes executing on the CPUs.

Project 4 CS 2200 – Systems and Networks Fall 2023

    Problem 0: The Ready Queue

We have provided simple implementations of queue t, enqueue(), dequeue(), and is empty() in student.c. The struct you have to implement will serve as your ready queue, and you should be using these helper func-tions to add and remove processes from the ready queue in the problems to follow.

2.1 Provided Queue

    The queue is backed by a linked list with each PCB acting as a node. There is a field in the PCB, next, which you may use to build linked lists of PCBs.

    enqueue() will add a process to the ready queue at the appropriate location.

    dequeue() will remove a process at the head of the ready queue and return a pointer to that process.

    NOTE: When using the ready queue helper functions in the following problems, make sure to call them in a thread-safe manner. Read up on how to use mutex locks and lock/unlock the mutex for the ready queue when you call these functions. You might need to modify the dequeue() function to support priority scheduling in Problem 3.

Project 4 CS 2200 – Systems and Networks Fall 2023

    Problem 1: FCFS Scheduler

Implement the CPU scheduler using the FCFS scheduling algorithm. You may do this however you like, however, we suggest the following:

    Implement the yield(), wake up(), and terminate() handlers. in student.c.

Checkout the hints in section 3.3, and note that preempt() is not necessary for this stage of the project.

    Implement idle().

idle() must wait on a condition variable that is signalled whenever a process is added to the ready queue.

    Implement schedule().

schedule() should extract the first process in the ready queue, then call context switch() to select the process to execute. If there are no runnable processes, schedule() should call context switch() with a NULL pointer as the PCB to execute the idle process.

3.1 Hints

    Be sure to update the state field of the PCB in all the methods above. The library will read this field to generate the RUNNING (Ru), READY (Re), and WAITING (Wa) columns, and to print the statistics at the end of the simulation.

    Four of the five entry points into the scheduler (idle(), yield(), terminate(), and preempt()) should cause a new process to be scheduled on the CPU. In your handlers, be sure to call schedule(), which will select a runnable process, and then call context switch(). When these four functions return, the library will simulate the execution of the process selected by context switch().

    context switch() takes a timeslice parameter, which is used for preemptive scheduling algorithms. Since FCFS is non-preemptive, use -1 for this parameter to give the process an infinite timeslice.

    Make sure to use the helper functions in a thread-safe manner when adding and removing processes from the ready queue!

    The current[] array should be used to keep track of the process currently executing on each CPU. Since this array is accessed by multiple CPU threads, it must be protected by a mutex. current mutex has been provided for you.

Project 4 CS 2200 – Systems and Networks Fall 2023

    Problem 2: Round-Robin Scheduler

Add Round-Robin scheduling functionality to your code. You should modify main() to add a command line option, -r, which selects the Round-Robin scheduling algorithm, and accepts a parameter, the length of the timeslice. For this project, timeslices are measured in tenths of seconds. E.g.:

./os-sim <# CPUs> -r 5

should run a Round-Robin scheduler with timeslices of 500 ms. While:

./os-sim <# of CPUs>

should continue to run a FCFS scheduler. Note: you can use getopt(), which we used earlier in the semester or just parse the command line arguments passed into main using if statements.

Implement preempt().

preempt() should place the currently running process back in the ready queue, and call schedule() to select a new runnable process.

To specify a timeslice when scheduling a process, use the timeslice parameter of context switch(). The simulator will simulate a timer interrupt to preempt the process and call your preempt() handler if the process executes on the CPU for the length of the timeslice without terminating or yielding for I/O.

    Problem 3: Preemptive Priority Scheduling with Aging

Add Priority with Aging scheduling to your code. Alter the provided enqueue() and/or dequeue() to support priority with aging. Modify main() to accept the -p parameter to select the Priority Scheduling with Aging algorithm. The command line argument will follow this format. ./os-sim <num CPUs> -p <age weight>. Take a look at your homework 4 if you are struggling with this.

Implement the function priority with age(). Each process has a base priority, however, we need to factor in its age to give us the processes’ functional priority. To do this, we must understand a few variables.

    current time is a running time function that tells us how long it has been since our simulator has been booted up. We can obtain this value by simply calling the function get current time().

    enqueue time is a value in the PCB that tells us when the process was put into the ready queue.

    age weight is an argument that is passed in from the command line. This value determines how much priority a process gains per unit age.

To calculate our functional priority use the equation

functional priority = base priority + (current time − enqueue time) ∗ age weight

We will calculate functional priority on every process in the ready queue and schedule the process with the highest priority. This means your ready queue does not have to stay in priority order. Choosing our process will take O(n), meaning we have to look at each process every time we choose one. (NOTE: Contrary to modern operating systems, higher number means higher priority.).

Project 4 CS 2200 – Systems and Networks Fall 2023

Figure 5: Example Ready Queue

The above ready queue is an example of our priority with aging algorithm. Our example simulator is at current time 15, has an age weight of .2, and four processes in its ready queue. Notice the following:

    Process 1 is earlier in the queue than process 2, yet process 2 has a higher functional priority.

    Process 4 has a higher base priority than process 3, yet process 3 has a higher functional priority.

    Process 2 and 4 have the same base priority, yet process 2 has a much higher functional priority.

    We would schedule process 2 since it has a best mix of old age and high priority.

    Problem 4: Short Answers

Please write your answers to the following questions in answers.txt.

6.1 FIFO Scheduler

Run your OS simulation with 1, 2, and 4 CPUs. Compare the total execution time of each. Is there a linear relationship between the number of CPUs and total execution time? Why or why not? Keep in mind that the execution time refers to the simulated execution time.

6.2 Round-Robin Scheduler

Run your Round-Robin scheduler with timeslices of 800ms, 600ms, 400ms, and 200ms. Use only one CPU for your tests. Compare the statistics at the end of the simulation. Is there a relationship between the total waiting time and timeslice length? If so, what is it? In contrast, in a real OS, the shortest timeslice possible is usually not the best choice. Why not?

6.3 Preemptive Priority Scheduler

Priority schedulers can sometimes lead to starvation among processes with lower priority. What is a way that operating systems can mitigate starvation in a priority scheduler?

6.4 Priority Inversion

Project 4 CS 2200 – Systems and Networks Fall 2023

to come in and starve the window manager process. The starvation of the window manager will also mean the starvation of P1 (the process with high priority), since P1 is waiting for the window manager to finish running.

If we want to keep our non-preemptive priority scheduler, what edits can we make to our scheduler to ensure that the P1 can finish its execution before any of the medium priority processes finish their execution? Explain in detail the changes you would make.

    The Gradescope Environment

You will be submitting files to Gradescope, where they will be tested in a VM setup that runs through Gradescope. The specifications of this VM are that it runs Ubuntu 20.04 LTS (64-bit) and gcc 9.3.0, and so we expect that your files can run in such a setup. This means thatwhen you are running your project locally, you will want to ensure you are using a VM/some setup that runs Ubuntu 20.04 LTS (64-bit) and gcc 9.3.0; this way, you can ensure that what occurs locally is what will occur when you submit to Gradescope.

IMPORTANT: Since we are dealing with different threads of execution, the result of each run in the simulation will be different. As a result, our gradescope autograder will accept a range of results for your simulation. In past semesters, we found that the range will start to increase the more computations you do in your enqueue/dequeue methods. If you find that you aren’t passing the autograder but you believe that your code is right, it is likely that you need to trim down your enqueue/dequeue methods. We are planning to start our autograder with a tighter acceptable range, and if required, we will increase it.

    Deliverables

NOTE: You need to upload student.c, and answers.txt to Gradescope, and an autograder will run to check if your scheduler is working. The autograder might take a couple of minutes to run. Remember to upload student.c, and answers.txt for every submission as your last submission would be one we will grade.

Keep your answers detailed enough to cover the question, including support from simulator results if appro-priate. Don’t write a book; but if you’re not sure about an answer, give us more information.

Project 4 CS 2200 – Systems and Networks Fall 2023

    How to Run / Debug Your Code – Debugging deadlocks and synchronization errors

9.1 Running

We have provided a Makefile that will run gcc for you. To compile your code with no optimizations (which you should do while developing, it will make debugging easier) and test with the FCFS algorithm and one CPU, run:

    make debug

    . / os−sim 1

To run the other algorithms, run with the flags you implemented for round robin and priority. Remember that round robin requires you to enter a time slice.

In case you encounter difficulties with Project 4 and are uncertain about the direction to take, various resources are available to assist you.

9.2 GDB

Let us investigate how to debug deadlocks through a basic example:

Project 4 CS 2200 – Systems and Networks Fall 2023

As anticipated, the program continues to remain unresponsive when run within the gdb environment. To interrupt the program, press Ctrl + c. To analyze the various threads associated with the program, utilize the ”info threads” command within gdb, which provides detailed information regarding each active thread:

Upon examining the running threads using the ”info threads” command within gdb, we can observe that threads 2 and 3, which were created within the main() function, are currently situated within the lll lock wait function. To obtain the backtrace of these threads, we can use the ”thread apply all” command along with the ”backtrace” command, which can be abbreviated as ”t a a bt”.

Project 4 CS 2200 – Systems and Networks Fall 2023

By switching to thread 3 within gdb, we can identify the precise line of code where it has become deadlocked. Once we have identified the problematic line, we can utilize gdb’s features, such as printing values or switching stack frames, to investigate further and gain a better understanding of the issue at hand. Read the gdb thread documentation here for more information.

9.3 Valgrind (Helgrind or DRD)

What about issues when accessing a shared resource? Valgrind has some handy tools to help detect such problems. You may use your 2110 docker or just look up ”valgrind installation” online for download instruc-tions. Let’s modify the code from the previous section:

Project 4 CS 2200 – Systems and Networks Fall 2023

Upon executing Valgrind’s tool Helgrind, we can observe that it has successfully identified an issue within the program where thread2 is accessing a shared variable without acquiring a corresponding lock. Additionally, DRD (another tool within Valgrind) also provides comparable output, albeit with fewer error lines. It is essential to rectify these issues to ensure proper synchronization and avoid potential data race conditions. To compare, here is the same program run with DRD (’valgrind tool=drd <program>’):

Valgrind and DRD are also able to debug other types of synchronization errors. You can read the documen-tation about Helgrind here and DRD here.

Credit to this video from an old class.
