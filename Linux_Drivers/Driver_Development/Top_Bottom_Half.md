# LDD3 — SoftIRQs, Tasklets & Bottom Halves

 Below is a GitHub-ready Markdown note based on the LDD3 sections you referenced, with the old LDD3 APIs clearly separated from the modern Linux-kernel mental model.

````
# Linux Device Drivers — SoftIRQs, Tasklets & Bottom Halves

> Based on **Linux Device Drivers, 3rd Edition (LDD3)**, especially Chapter 7 and Chapter 10.
>
> The code/API shown in LDD3 is from an older Linux kernel. Modern kernels have changed several APIs, but the **core concepts and execution rules are extremely important and still form the foundation for understanding interrupt/deferred-work processing**.

---

# 1. The Big Problem: Interrupts Are Fast, But Work Can Be Slow

Imagine a network card receives a packet.

The hardware says:

```text
"Hey CPU! I received something!"
````

 The CPU immediately enters the interrupt handler.

 The problem:

```
Interrupt Handler
      |
      |---- read device data
      |---- process packet
      |---- wake processes
      |---- allocate memory
      |---- do lots of work
      |
      +---- finally return
```

 This is bad.

 Why?

 Because while the interrupt handler is doing too much work, the system is spending too much time handling the interrupt.

 The fundamental rule is:

 > **Do the minimum necessary work in the interrupt handler, and defer the expensive work to a later context.**

 This is the reason for:

 - Top half
- Bottom half
- SoftIRQ
- Tasklet
- Workqueue

---

 # 2\. The Most Important Mental Model

 Remember this picture:

```
                    HARDWARE
                       |
                       | interrupt
                       v
              +-------------------+
              |    TOP HALF       |
              |  Hard IRQ Handler |
              +-------------------+
                       |
                       | "I'll do the rest later"
                       v
              +-------------------+
              |   BOTTOM HALF     |
              |                   |
              |  SoftIRQ/Tasklet  |
              |       OR          |
              |    Workqueue      |
              +-------------------+
                       |
                       v
                 More processing
```

 The top half should be:

```
FAST
SHORT
NON-SLEEPING
```

 The bottom half can perform the remaining work.

---

 # 3\. Why Split Interrupt Processing?

 Suppose an interrupt handler takes 10 ms.

 During those 10 ms, the CPU is busy dealing with interrupt-related work.

 Instead:

```
Interrupt arrives
      |
      v
Top half
  |
  | save important information
  | acknowledge device
  | schedule bottom half
  |
  +---- return quickly
            |
            v
       Bottom half
       does the heavy work
```

 Now the interrupt handler can finish quickly.

 This improves:

 - interrupt latency
- system responsiveness
- ability to handle multiple interrupts
- overall throughput

---

 # 4\. Top Half

 The **top half** is the actual interrupt handler registered with `request_irq()`.

 Conceptually:

```
irq_handler()
{
    read_device_status();

    save_important_data();

    schedule_bottom_half();

    return IRQ_HANDLED;
}
```

 The top half should avoid doing lengthy work.

 Think:

 > **Top half = "Capture the event and schedule the rest."**

---

 # 5\. Bottom Half

 The **bottom half** performs work that does not need to happen immediately.

 For example:

```
Top half:
    "Packet arrived."

Bottom half:
    "Now let's actually process the packet."
```

 A bottom half can be implemented using mechanisms such as:

 - SoftIRQ
- Tasklet
- Workqueue

 The important distinction is:

```
Tasklet
    |
    +-- runs in softirq context
    +-- cannot sleep

Workqueue
    |
    +-- runs in process context
    +-- can sleep
```

 This distinction is one of the most important things to remember.

---

 # 6\. SoftIRQ

 A **softirq** is a mechanism used by the Linux kernel to execute deferred work in a software interrupt context.

 The key idea:

```
Hard IRQ
   |
   | schedule deferred work
   v
SoftIRQ
```

 A softirq is NOT a hardware interrupt.

 It is software-generated deferred work.

---

 # 7\. Hard IRQ vs SoftIRQ

 | Property | Hard IRQ | SoftIRQ |
| --- | --- | --- |
| Triggered by | Hardware/device | Kernel/software |
| Purpose | Respond immediately to hardware | Defer work |
| Can sleep? | No | No |
| Runs in interrupt context? | Yes | Yes |
| Should be fast? | Yes | Yes |
| Can run concurrently on multiple CPUs? | Yes | Yes |

The important point:

 > **SoftIRQ is still atomic/interrupt-like context.**

 Therefore:

```
SoftIRQ
   |
   +-- cannot sleep
   +-- cannot call sleeping functions
   +-- must be careful with locking
```

---

 # 8\. Why Can't SoftIRQ Sleep?

 A softirq does not run like a normal user process.

 There is no normal process context available for:

```
schedule();
```

 or other operations that put the current execution context to sleep.

 Therefore:

```
SoftIRQ
   |
   +-- NO sleep
   +-- NO blocking
   +-- NO waiting for userspace
```

 This gives us a useful rule:

 > **If your deferred work might need to sleep, don't use a tasklet/softirq. Use a workqueue.**

---

 # 9\. Tasklets

 A tasklet is a convenient mechanism built on top of the softirq mechanism.

 The simplest mental model:

```
Tasklet = convenient deferred-work mechanism running in softirq context
```

 Therefore:

```
Tasklet
   |
   +-- runs later
   +-- runs in softirq context
   +-- cannot sleep
   +-- is useful for short deferred work
```

---

 # 10\. Tasklet Declaration

 LDD3 shows:

```
#include <linux/interrupt.h>

DECLARE_TASKLET(name, func, data);
```

 Example:

```
void short_do_tasklet(unsigned long data)
{
    /* deferred work */
}

DECLARE_TASKLET(short_tasklet, short_do_tasklet, 0);
```

 Here:

```
short_tasklet
      |
      +---- function = short_do_tasklet
      |
      +---- data = 0
```

 When the tasklet executes:

```
short_do_tasklet(0);
```

---

 # 11\. Tasklet Initialization

 LDD3 also describes:

```
void tasklet_init(
    struct tasklet_struct *t,
    void (*func)(unsigned long),
    unsigned long data
);
```

 This is useful when the tasklet structure already exists, for example because it was dynamically allocated.

 Conceptually:

```
struct tasklet_struct
        |
        +---- function
        +---- data
        +---- state
```

 `tasklet_init()` initializes that structure.

---

 # 12\. Disabled Tasklet

 LDD3:

```
DECLARE_TASKLET_DISABLED(name, func, data);
```

 This creates a tasklet initially disabled.

 It can later be enabled:

```
tasklet_enable(&tasklet);
```

---

 # 13\. Enabling and Disabling Tasklets

 LDD3 provides:

```
void tasklet_disable(struct tasklet_struct *t);
void tasklet_disable_nosync(struct tasklet_struct *t);
void tasklet_enable(struct tasklet_struct *t);
```

 Think about the difference between:

```
disable()
```

 and

```
disable_nosync()
```

---

 ## 13.1 tasklet\_disable()

 Conceptually:

```
"Disable this tasklet,
and make sure it isn't currently executing."
```

 On SMP systems, if another CPU is currently executing the tasklet, `tasklet_disable()` can wait for it to finish.

 So:

```
CPU 0                         CPU 1

                              tasklet running
                                   |
disable(tasklet) -----------------+
                                   |
                              finish tasklet
                                   |
disable returns
```

---

 ## 13.2 tasklet\_disable\_nosync()

 This disables the tasklet without waiting for a currently executing instance to finish.

 Therefore:

```
tasklet_disable()
        |
        +-- disable
        +-- synchronize/wait if needed

tasklet_disable_nosync()
        |
        +-- disable
        +-- don't wait
```

---

 # 14\. Disable/Enable Must Match

 If you disable a tasklet:

```
tasklet_disable(&tasklet);
```

 you eventually need:

```
tasklet_enable(&tasklet);
```

 Think of it like a counter:

```
disable()
disable()
disable()

       ↓

disabled count = 3

enable()
enable()
enable()

       ↓

disabled count = 0
       |
       v
enabled
```

 The important conceptual rule:

 > **Every disable needs a matching enable.**

---

 # 15\. Scheduling a Tasklet

 The main function:

```
tasklet_schedule(&tasklet);
```

 means:

 > "Run this tasklet later."

 Example:

```
irq_handler()
{
    /* do minimal interrupt work */

    tasklet_schedule(&my_tasklet);

    return IRQ_HANDLED;
}
```

 The sequence is:

```
Hardware interrupt
       |
       v
Hard IRQ handler
       |
       +---- capture data
       |
       +---- tasklet_schedule()
       |
       +---- return
              |
              v
         tasklet runs
```

---

 # 16\. Very Important: Tasklet Scheduling Is NOT Cumulative

 This is one of the easiest things to misunderstand.

 Suppose:

```
tasklet_schedule(&my_tasklet);
tasklet_schedule(&my_tasklet);
tasklet_schedule(&my_tasklet);
tasklet_schedule(&my_tasklet);
```

 You might think:

```
run
run
run
run
```

 But that is NOT how tasklet scheduling works.

 The tasklet is effectively marked as:

```
"already scheduled"
```

 So multiple scheduling requests before it actually runs do not create multiple executions.

 Conceptually:

```
schedule
   |
   v
[ scheduled ]

schedule
   |
   v
[ already scheduled ]

schedule
   |
   v
[ already scheduled ]

schedule
   |
   v
[ already scheduled ]

       |
       v

     RUN ONCE
```

 Therefore:

 > **Tasklet scheduling requests can collapse into one execution.**

---

 # 17\. The Consequence: Don't Lose Information

 This creates a very important driver-design requirement.

 Suppose 100 interrupts arrive before the tasklet executes.

 The tasklet may run only once.

 Therefore the driver must preserve enough information to know:

```
"100 interrupts happened."
```

 For example:

```
interrupt_count++;
tasklet_schedule(&tasklet);
```

 The tasklet can later examine:

```
interrupt_count
```

 and process all the work represented by those interrupts.

 This is a very important pattern:

```
TOP HALF

interrupt arrives
      |
      +---- save information
      |
      +---- increment count
      |
      +---- schedule tasklet

BOTTOM HALF

tasklet runs
      |
      +---- inspect saved information
      |
      +---- process accumulated work
```

---

 # 18\. Tasklets Do Not Run in Parallel With Themselves

 Suppose:

```
my_tasklet
```

 is scheduled.

 Linux will not execute the same tasklet instance simultaneously on two CPUs.

 So:

```
CPU 0:
    my_tasklet running

CPU 1:
    my_tasklet cannot simultaneously run
```

 This is a useful property.

 But don't confuse it with:

 > "Tasklets are globally serialized."

 They are NOT.

---

 # 19\. Different Tasklets Can Run in Parallel

 Suppose we have:

```
tasklet_A
tasklet_B
tasklet_C
```

 On an SMP system:

```
CPU 0                  CPU 1

tasklet_A              tasklet_B
   |                       |
   |                       |
   +                       +
```

 They can execute concurrently.

 Therefore, if two tasklets access shared data:

```
shared_data
```

 you may need synchronization.

---

 # 20\. Tasklet + Interrupt Handler Can Race

 Another important point.

 Suppose:

```
CPU 0

interrupt handler
      |
      | schedule tasklet
      |
      v
tasklet
```

 The tasklet does not start executing before the scheduling interrupt handler has completed on that CPU.

 However, another interrupt can occur while the tasklet is executing.

 Therefore:

```
Tasklet
   |
   +------ running
             |
             +---- interrupt arrives
                       |
                       v
                  interrupt handler
```

 Now both pieces of code may access shared data.

 Therefore:

 > **You may still need locking between the interrupt handler and its tasklet.**

---

 # 21\. Tasklet Execution Context

 This is extremely important:

```
Tasklet
   |
   v
SoftIRQ context
```

 Therefore:

```
CAN:
    perform atomic operations
    access kernel data structures appropriate for atomic context
    execute quickly

CANNOT:
    sleep
    block
    wait for something that sleeps
```

 Remember:

 > **Tasklet = SoftIRQ context = NO SLEEP**

---

 # 22\. High-Priority Tasklets

 LDD3 describes:

```
tasklet_schedule(&tasklet);
```

 and:

```
tasklet_hi_schedule(&tasklet);
```

 The second schedules a high-priority tasklet.

 Conceptually:

```
SoftIRQ processing

       |
       +---- high-priority tasklets
       |
       +---- normal tasklets
```

 High-priority tasklets are handled before normal tasklets.

---

 # 23\. Killing a Tasklet

 LDD3:

```
tasklet_kill(&tasklet);
```

 Conceptually:

 > "Make sure this tasklet is no longer scheduled/running."

 This is especially important during cleanup/module removal.

 Imagine:

```
module_exit()
     |
     +---- free memory used by tasklet
```

 But what if:

```
tasklet is still running
```

 and then it accesses that freed memory?

 That would be a serious bug.

 Therefore cleanup must ensure deferred work has stopped before freeing its resources.

---

 # 24\. The Classic Driver Lifetime Problem

 Imagine:

```
driver memory
     |
     +---- tasklet uses this memory
```

 Then:

```
module unload
     |
     +---- free driver memory
```

 But the tasklet is still scheduled.

 Later:

```
tasklet runs
     |
     +---- accesses freed memory
```

 BAD.

 Therefore:

```
STOP/CANCEL DEFERRED WORK
          |
          v
WAIT FOR IT TO FINISH
          |
          v
FREE MEMORY
```

 This principle is much more important than memorizing individual APIs.

---

 # 25\. Tasklet Summary

 Remember tasklets using this sentence:

 > **Tasklet = small deferred job that runs in softirq context and therefore cannot sleep.**

 Properties:

```
Tasklet
  |
  +-- deferred
  +-- softirq context
  +-- cannot sleep
  +-- same tasklet instance doesn't execute concurrently with itself
  +-- different tasklets can run concurrently on SMP
  +-- scheduling is not cumulative
  +-- good for short atomic work
```

---

 # 26\. Workqueues

 Now we come to the alternative:

```
Workqueue
```

 A workqueue also performs deferred work.

 But there is a huge difference:

```
Tasklet       -> atomic/softirq context
Workqueue     -> process context
```

 Therefore:

```
Tasklet
   |
   +---- cannot sleep

Workqueue
   |
   +---- CAN sleep
```

 This is the key distinction.

---

 # 27\. Mental Model of a Workqueue

 Think:

```
Interrupt
    |
    v
Top half
    |
    | queue work
    v
+-------------------+
|    Workqueue      |
|                   |
|   Worker thread   |
+-------------------+
          |
          v
     work function
```

 The work function executes in the context of a worker thread.

 Therefore it has process context.

---

 # 28\. Why Workqueues Exist

 Suppose your deferred operation needs to:

```
wait
sleep
block
perform potentially long processing
```

 A tasklet is inappropriate.

 Instead:

```
Interrupt
    |
    v
schedule work
    |
    v
Worker thread
    |
    +---- can sleep
    +---- can block
    +---- perform longer work
```

 So remember:

 > **Need to sleep? Think workqueue.**

---

 # 29\. Workqueue Structures

 LDD3 introduces:

```
struct workqueue_struct;
struct work_struct;
```

 Think:

```
workqueue_struct
        |
        +---- collection/queue of work

work_struct
        |
        +---- one unit of deferred work
```

---

 # 30\. Creating a Workqueue

 LDD3:

```
create_workqueue(const char *name);
```

 This creates a workqueue with worker threads associated with CPUs.

 LDD3 also describes:

```
create_singlethread_workqueue(const char *name);
```

 which creates a workqueue with a single worker.

 Destroy:

```
destroy_workqueue(queue);
```

---

 # 31\. Workqueue Initialization

 LDD3:

```
DECLARE_WORK(name, function, data);
```

 or:

```
INIT_WORK(work, function, data);
```

 Conceptually:

```
work_struct
      |
      +---- function to execute
      |
      +---- data
```

 Then later:

```
queue work
      |
      v
worker executes function
```

---

 # 32\. Queueing Work

 LDD3:

```
queue_work(queue, &work);
```

 This means:

 > "Put this work item onto this workqueue."

 The worker thread will execute it later.

---

 # 33\. Delayed Work

 LDD3:

```
queue_delayed_work(
    queue,
    &work,
    delay
);
```

 This means:

```
Don't execute immediately.

Wait for delay.

Then execute the work.
```

 Conceptually:

```
queue_delayed_work()
        |
        v
      TIMER
        |
        | delay expires
        v
     WORKQUEUE
        |
        v
    work function
```

---

 # 34\. Canceling Work

 LDD3:

```
cancel_delayed_work(&work);
```

 This attempts to remove delayed work from the queue.

 Again, the underlying idea is:

 > **Before destroying resources, make sure deferred work cannot unexpectedly access them.**

---

 # 35\. Flushing a Workqueue

 LDD3:

```
flush_workqueue(queue);
```

 The purpose is to wait until work queued on that workqueue has finished executing.

 Think:

```
flush_workqueue()
       |
       v
"Wait until queued work is finished."
```

 This is extremely useful during cleanup.

---

 # 36\. Shared/System Workqueue

 LDD3 also describes:

```
schedule_work(&work);
```

 This places work onto the shared system workqueue.

 Conceptually:

```
your driver
    |
    +---- schedule_work()
             |
             v
      shared workqueue
             |
             v
       worker thread
             |
             v
        your function
```

 You don't necessarily need to create your own workqueue for every small piece of deferred work.

---

 # 37\. Delayed Shared Work

 LDD3:

```
schedule_delayed_work(
    &work,
    delay
);
```

 This is the shared-workqueue equivalent of delayed work.

---

 # 38\. Workqueue vs Tasklet

 This table is worth memorizing.

 | Feature | Tasklet | Workqueue |
| --- | --- | --- |
| Runs in | SoftIRQ context | Process context |
| Can sleep? | **NO** | **YES** |
| Can block? | **NO** | **YES** |
| Good for | Short atomic work | Work that may sleep |
| Latency | Generally lower | Generally higher |
| Worker thread | No | Yes |
| Same work item concurrent with itself? | No | Depends on workqueue semantics/API |
| Suitable for lengthy work? | No | Yes |

The most important row:

```
TASKLET     -> CANNOT SLEEP
WORKQUEUE   -> CAN SLEEP
```

---

 # 39\. Top Half + Tasklet

 The complete flow:

```
              HARDWARE
                  |
                  | interrupt
                  v
        +-------------------+
        |     TOP HALF      |
        |   IRQ handler     |
        +-------------------+
                  |
                  | save data
                  |
                  | tasklet_schedule()
                  |
                  v
        +-------------------+
        |     TASKLET       |
        |                   |
        |  softirq context  |
        +-------------------+
                  |
                  v
             process data
```

---

 # 40\. Top Half + Workqueue

 The alternative:

```
              HARDWARE
                  |
                  | interrupt
                  v
        +-------------------+
        |     TOP HALF      |
        |   IRQ handler     |
        +-------------------+
                  |
                  | save data
                  |
                  | schedule_work()
                  |
                  v
        +-------------------+
        |    WORKQUEUE      |
        |                   |
        |  worker thread    |
        +-------------------+
                  |
                  v
             process data
```

---

 # 41\. Why Not Just Do Everything in the IRQ Handler?

 Because the IRQ handler has strict constraints.

 Bad:

```
irq_handler()
{
    read_device();

    process_1000_packets();

    sleep();

    wait_for_hardware();

    do_expensive_calculation();

    return IRQ_HANDLED;
}
```

 Better:

```
irq_handler()
{
    read_device();

    save_required_information();

    schedule_work();

    return IRQ_HANDLED;
}
```

 Then:

```
worker()
{
    /* do the expensive work */
}
```

---

 # 42\. LDD3 "short" Driver Example

 LDD3's `short` driver demonstrates the concept.

 The top half receives the interrupt:

```
irqreturn_t short_tl_interrupt(...)
{
    /* Save interrupt information */

    do_gettimeofday(...);

    short_incr_tv(&tv_head);

    /* Schedule deferred processing */
    tasklet_schedule(&short_tasklet);

    /* Record interrupt */
    short_wq_count++;

    return IRQ_HANDLED;
}
```

 Notice how little the interrupt handler does.

 It:

```
1. Records information
2. Schedules tasklet
3. Records interrupt count
4. Returns
```

 That is the core top-half design.

---

 # 43\. The Tasklet Function

 The deferred function:

```
void short_do_tasklet(unsigned long unused)
{
    ...
}
```

 does the larger amount of work.

 It reads the information saved by the top half.

 Conceptually:

```
TOP HALF

tv_head
   |
   +---- new timestamp
   |
   +---- interrupt count
   |
   +---- schedule tasklet

TASKLET

tv_tail
   |
   +---- process saved timestamps
   |
   +---- write data to buffer
   |
   +---- wake readers
```

---

 # 44\. Why Does the Driver Keep `tv_head` and `tv_tail`?

 Think of the buffer as a queue.

```
                circular buffer

        +---------------------------+
        | timestamp | timestamp |   |
        +---------------------------+
              ^                 ^
              |                 |
           tv_tail            tv_head
```

 The top half adds information at the head.

 The bottom half consumes information from the tail.

 So:

```
Producer                  Consumer

TOP HALF                  TASKLET
   |                         |
   | write                   | read
   v                         v
+----------------------------------+
|      circular buffer             |
+----------------------------------+
```

 This is a very common kernel pattern.

---

 # 45\. Multiple Interrupts Before One Tasklet

 This is extremely important.

 Suppose:

```
Interrupt 1
Interrupt 2
Interrupt 3
Interrupt 4
Interrupt 5
```

 arrive quickly.

 The top half may execute five times:

```
IRQ 1 -> schedule tasklet
IRQ 2 -> schedule tasklet
IRQ 3 -> schedule tasklet
IRQ 4 -> schedule tasklet
IRQ 5 -> schedule tasklet
```

 But the tasklet may execute only once.

 Therefore the driver needs to preserve the five events somewhere:

```
interrupt_count = 5
```

 or:

```
5 timestamps stored in buffer
```

 Then the tasklet processes the accumulated information.

 This is a crucial design principle:

 > **Deferred work scheduling does not necessarily represent the number of events. The data structures must preserve the events.**

---

 # 46\. Tasklet Execution Timeline

 Imagine this:

```
TIME -------------------------------------------------->

IRQ       IRQ       IRQ
 |         |         |
 v         v         v
+---+     +---+     +---+
|IRQ|     |IRQ|     |IRQ|
+---+     +---+     +---+
  |         |         |
  +---------+---------+
            |
       tasklet scheduled
            |
            v
        +---------+
        | TASKLET |
        +---------+
            |
            v
        process ALL
        saved data
```

 Do not think:

```
3 interrupts = 3 tasklet executions
```

 Think:

```
3 interrupts = potentially ONE tasklet execution
                  that processes 3 events
```

---

 # 47\. Why Tasklets Are Safe From Self-Concurrency

 Suppose:

```
tasklet A
```

 is already running.

 Another CPU cannot simultaneously run that same tasklet instance.

 So:

```
CPU 0:
    tasklet A running

CPU 1:
    tasklet A cannot run simultaneously
```

 This makes some designs easier.

 But:

```
tasklet A
tasklet B
```

 can run simultaneously on different CPUs.

 Therefore shared data between different tasklets still requires synchronization.

---

 # 48\. SMP Changes Everything

 SMP means:

```
Symmetric MultiProcessing
```

 In simple terms:

```
CPU 0
CPU 1
CPU 2
CPU 3
```

 all execute kernel code.

 Therefore:

```
"Only one CPU exists"
```

 is an unsafe assumption.

 For example:

```
CPU 0                    CPU 1

interrupt handler        tasklet
      |                     |
      +---------+-----------+
                |
           shared data
```

 Both may access the same data.

 Therefore synchronization may be required.

---

 # 49\. Atomic Context

 Tasklets and softirqs execute in an environment where sleeping is forbidden.

 This is commonly referred to as:

```
atomic context
```

 The practical rule:

```
Atomic context
      |
      +-- DON'T SLEEP
      +-- DON'T BLOCK
```

 When writing kernel code, always ask:

 > "Can this function sleep?"

 If yes, don't call it from a tasklet/softirq/IRQ context.

---

 # 50\. A Useful Context Ladder

 Memorize this:

```
                    KERNEL EXECUTION CONTEXTS

                  +----------------------+
                  |   Process Context    |
                  |                      |
                  |   CAN SLEEP          |
                  |                      |
                  |   Workqueue          |
                  +----------------------+
                            ^
                            |
                    deferred work
                            |
                  +----------------------+
                  | Interrupt Context    |
                  |                      |
                  |   CANNOT SLEEP       |
                  |                      |
                  | Hard IRQ             |
                  | SoftIRQ / Tasklet    |
                  +----------------------+
```

 The exact modern kernel execution details are more nuanced, but this is an excellent foundational mental model.

---

 # 51\. The Golden Question

 Whenever you need to defer work, ask:

```
"Does this work need to sleep?"
```

 If:

```
NO
```

 then a softirq/tasklet-style mechanism may be appropriate.

 If:

```
YES
```

 then you need process context, such as a workqueue.

 Remember:

```
NO SLEEP  -> Tasklet/SoftIRQ style
CAN SLEEP -> Workqueue
```

---

 # 52\. Tasklet API Cheat Sheet — LDD3

```
DECLARE_TASKLET(name, func, data);
```

 Declare a tasklet.

```
DECLARE_TASKLET_DISABLED(name, func, data);
```

 Declare a disabled tasklet.

```
tasklet_init(&tasklet, func, data);
```

 Initialize an existing tasklet structure.

```
tasklet_schedule(&tasklet);
```

 Schedule normal tasklet execution.

```
tasklet_hi_schedule(&tasklet);
```

 Schedule high-priority tasklet execution.

```
tasklet_disable(&tasklet);
```

 Disable and synchronize with a running tasklet.

```
tasklet_disable_nosync(&tasklet);
```

 Disable without waiting for a currently running tasklet.

```
tasklet_enable(&tasklet);
```

 Enable the tasklet.

```
tasklet_kill(&tasklet);
```

 Ensure the tasklet is no longer active before cleanup.

````

---

# 53. Workqueue API Cheat Sheet — LDD3

```c
create_workqueue("name");
````

 Create a workqueue.

```
create_singlethread_workqueue("name");
```

 Create a single-threaded workqueue.

```
destroy_workqueue(queue);
```

 Destroy a workqueue.

```
DECLARE_WORK(name, function, data);
```

 Declare work.

```
INIT_WORK(&work, function, data);
```

 Initialize work.

```
queue_work(queue, &work);
```

 Queue work.

```
queue_delayed_work(queue, &work, delay);
```

 Queue delayed work.

```
cancel_delayed_work(&work);
```

 Cancel delayed work.

```
flush_workqueue(queue);
```

 Wait for queued work to finish.

```
schedule_work(&work);
```

 Schedule work on the shared/system workqueue.

```
schedule_delayed_work(&work, delay);
```

 Schedule delayed work on the shared/system workqueue.

```
flush_scheduled_work();
```

 Flush the shared/system workqueue.

````

---

# 54. Important Modern-Kernel Warning

LDD3 was written for an old Linux kernel.

Therefore:

```text
LDD3 API != current Linux API
````

 Do NOT blindly copy LDD3 code into a modern kernel driver.

 For example, old interfaces and types shown in LDD3 may have changed or disappeared.

 The concepts are still extremely valuable:

```
Hard IRQ
   |
   v
Deferred work
   |
   +---- SoftIRQ
   |
   +---- Tasklet
   |
   +---- Workqueue
```

 But always check the current kernel source/documentation before writing modern driver code.

---

 # 55\. The Most Important Difference

 If you remember only one table from this chapter, remember this:

 | Mechanism | Context | Can Sleep? | Main Idea |
| --- | --- | --- | --- |
| Hard IRQ | Interrupt context | ❌ | Respond immediately |
| SoftIRQ | Softirq/atomic context | ❌ | Deferred kernel work |
| Tasklet | Softirq context | ❌ | Convenient small deferred work |
| Workqueue | Process context | ✅ | Deferred work that may sleep |

---

 # 56\. Top Half vs Bottom Half

 Another table worth memorizing:

 | Top Half | Bottom Half |
| --- | --- |
| Runs immediately after interrupt | Runs later |
| Must be very fast | Can perform more work |
| Interrupt context | Depends on mechanism |
| Cannot sleep | Tasklet: cannot sleep |
| Captures important information | Processes deferred information |
| Schedules bottom half | Performs remaining work |

---

 # 57\. One Complete Example

 Suppose a network device receives a packet.

 ## Step 1 — Hardware

```
NIC receives packet
       |
       v
Hardware generates IRQ
```

 ## Step 2 — Top Half

```
IRQ handler
    |
    +---- acknowledge interrupt
    |
    +---- retrieve minimal information
    |
    +---- store packet information
    |
    +---- schedule deferred processing
    |
    +---- return
```

 ## Step 3 — Deferred Processing

 If the work is short and cannot sleep:

```
Tasklet/SoftIRQ
       |
       +---- process data
```

 If the work can sleep:

```
Workqueue
       |
       +---- process data
       +---- sleep/block if necessary
```

---

 # 58\. A Memory Trick

 Remember the word:

 # **H-T-B-W**

```
H = Hardware
T = Top half
B = Bottom half
W = Work
```

 The flow:

```
Hardware
   |
   v
Top Half
   |
   v
Bottom Half
   |
   +---- Tasklet/SoftIRQ
   |
   +---- Workqueue
```

 But an even better memory trick is:

 # **FAST → DEFER → CHOOSE**

```
FAST
  |
  | IRQ handler
  v

DEFER
  |
  | schedule work
  v

CHOOSE
  |
  +---- Cannot sleep? --> Tasklet/SoftIRQ
  |
  +---- Can sleep? ----> Workqueue
```

---

 # 59\. Interview-Level Questions

 ## Q1. Why do we need bottom halves?

 Because interrupt handlers should finish quickly. Expensive processing is deferred so the interrupt handler can return quickly.

---

 ## Q2. Can a tasklet sleep?

 **No.**

 A tasklet executes in softirq/atomic context.

---

 ## Q3. Can a workqueue function sleep?

 **Yes.**

 A workqueue function executes in process context.

---

 ## Q4. Can two CPUs execute the same tasklet simultaneously?

 For the same tasklet instance, **no**.

 But different tasklets can execute concurrently on different CPUs.

---

 ## Q5. If I schedule the same tasklet five times, will it execute five times?

 Not necessarily.

 Tasklet scheduling is not cumulative. Multiple scheduling requests before execution can collapse into a single execution.

 Therefore the driver must preserve event information separately.

---

 ## Q6. Can a tasklet and an interrupt handler access the same data?

 Yes.

 Therefore they may need synchronization.

---

 ## Q7. Why not perform everything inside the interrupt handler?

 Because lengthy interrupt handling increases interrupt latency and can prevent the system from responding quickly to other events.

---

 ## Q8. Why use a workqueue instead of a tasklet?

 Use a workqueue when the deferred work may need to sleep or block.

---

 ## Q9. Why use a tasklet instead of a workqueue?

 When the work is short and must execute in atomic/softirq context.

---

 ## Q10. Why do we need to cancel/flush deferred work during cleanup?

 Because deferred code may still execute after the driver/module has freed the memory it uses.

 That can result in use-after-free bugs.

---

 # 60\. The Ultimate Mental Picture

 If you remember only one diagram, remember this:

```
                         HARDWARE
                            |
                            | IRQ
                            v
                 +----------------------+
                 |       TOP HALF       |
                 |    Hard IRQ Handler  |
                 |                      |
                 |  "Do minimum work"   |
                 +----------------------+
                            |
                            | defer
                            v
                    +---------------+
                    | BOTTOM HALF   |
                    +---------------+
                       /           \
                      /             \
                     v               v
              +-----------+    +-------------+
              |  TASKLET  |    | WORKQUEUE   |
              +-----------+    +-------------+
              | SoftIRQ   |    | Process     |
              | context   |    | context     |
              |           |    |             |
              | NO SLEEP  |    | CAN SLEEP   |
              +-----------+    +-------------+
```

 # 61\. Final Rules to Never Forget

 ### Rule 1

 > **Interrupt handler should be FAST.**

 ### Rule 2

 > **Don't do expensive work in the hard IRQ handler.**

 ### Rule 3

 > **Defer work when possible.**

 ### Rule 4

 > **Tasklet runs in softirq context.**

 ### Rule 5

 > **Tasklet CANNOT sleep.**

 ### Rule 6

 > **Workqueue runs in process context.**

 ### Rule 7

 > **Workqueue CAN sleep.**

 ### Rule 8

 > **Same tasklet instance does not execute concurrently with itself.**

 ### Rule 9

 > **Different tasklets can execute concurrently on SMP.**

 ### Rule 10

 > **Tasklet scheduling is not cumulative.**

 ### Rule 11

 > **Therefore, never assume one scheduled tasklet means one event.**

 ### Rule 12

 > **Store enough information so the deferred function can determine how much work needs to be done.**

 ### Rule 13

 > **During cleanup, stop/flush/kill deferred work before freeing the memory it uses.**

 ### Rule 14

 > **If the work needs to sleep → think WORKQUEUE.**

 ### Rule 15

 > **If the work is short and cannot sleep → TASKLET/SOFTIRQ-style deferred execution.**

---

 # 62\. One-Line Summary

```
HARD IRQ = respond NOW

TASKLET/SOFTIRQ = do small deferred work LATER, but DON'T SLEEP

WORKQUEUE = do deferred work LATER, and you CAN SLEEP
```

 And the complete idea:

```
              INTERRUPT
                  |
                  v
          +---------------+
          |   TOP HALF    |
          |    FAST       |
          +---------------+
                  |
                  | defer
                  v
          +---------------+
          | BOTTOM HALF   |
          +---------------+
             /         \
            /           \
           v             v
       TASKLET       WORKQUEUE
       SOFTIRQ       PROCESS CTX
       NO SLEEP      CAN SLEEP
```

 > **The entire purpose of this architecture is simple:**
>
>  **React quickly to the hardware, then do the rest of the work later in the appropriate execution context.**

```

### One important modern-Linux note

For learning LDD3, the above mental model is exactly the right foundation. For **new Linux kernel code**, however, tasklets have been deprecated/removed from modern development in favor of other deferred-work mechanisms, so don't treat the LDD3 tasklet API as a recommendation for new drivers. The **execution-context distinction**—especially **atomic context vs process context, and "can this code sleep?"**—is the part you should permanently internalize.
```
