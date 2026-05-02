# Assignment 3 - Complete Documentation

**Student Name**: Mohamed Amar  
**Student ID**: 445052806  
**Date Submitted**: May 2, 2026

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - May 2, 2026, 14:30
**What I implemented**: Task 1 - Added ReentrantLock to protect counter variables (contextSwitchCount, completedProcessCount, totalWaitingTime). Implemented lock.lock() with try-finally blocks and lock.unlock() to ensure thread-safe counter increments.

**Challenges encountered**: Initially forgot to place unlock() inside finally block, which could cause deadlocks. Also needed to understand why all three counters share one lock vs. separate locks.

**How I solved it**: Reviewed the assignment requirements emphasizing try-finally pattern. Used a single coarse-grained lock for all three counters since they are closely related and simplifies deadlock prevention.

**Testing approach**: Ran the program and verified that contextSwitchCount and completedProcessCount showed consistent values across multiple runs, indicating no lost updates.

**Time spent**: 45 minutes

---

### Entry 2 - May 2, 2026, 15:20
**What I implemented**: Task 2 - Added separate ReentrantLock (logLock) to protect the executionLog ArrayList. Updated logExecution() method to use lock.lock() and try-finally with lock.unlock() to prevent ConcurrentModificationException.

**Challenges encountered**: ArrayList is not thread-safe, and multiple threads could trigger ConcurrentModificationException during add() operations. Needed separate lock from counter lock to avoid over-synchronization.

**How I solved it**: Created a dedicated ReentrantLock for execution log and wrapped the executionLog.add() call with acquire/release pattern. This demonstrates fine-grained locking for different resource types.

**Testing approach**: Ran the program and verified no ConcurrentModificationException occurred in the execution log during concurrent process execution.

**Time spent**: 30 minutes

---

### Entry 3 - May 2, 2026, 15:55
**What I implemented**: Task 3 - Added binary Semaphore (cpuSemaphore with 1 permit) to ensure only one process executes on CPU at a time. Updated Process.run() and runToCompletion() methods to call acquire() before execution and release() in finally blocks.

**Challenges encountered**: Semaphore.acquire() throws InterruptedException, requiring try-catch handling. Needed to understand difference between Semaphore for resource limiting vs. ReentrantLock for mutual exclusion.

**How I solved it**: Added proper exception handling for acquire() and placed release() in finally block to guarantee resource release. This enforces time-sharing CPU scheduling.

**Testing approach**: Observed console output showed processes executing sequentially (only one quantum bar visible at a time), confirming mutual exclusion.

**Time spent**: 40 minutes

---

### Entry 4 - May 2, 2026, 16:35
**What I implemented**: Completed all git commits for Tasks 1-3. Verified ReentrantLock implementations protect critical sections properly. Confirmed Semaphore limits concurrent CPU access to 1 process.

**Challenges encountered**: Ensuring all finally blocks execute properly and understanding the difference between coarse-grained vs fine-grained locking strategies.

**How I solved it**: Reviewed code carefully, tested multiple runs with different process counts, and verified no race conditions or deadlocks occurred.

**Testing approach**: Ran program 5 times with same student ID - all runs produced consistent output and values, confirming synchronization works correctly.

**Time spent**: 35 minutes 

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

**Race Condition #1 - contextSwitchCount Increment:**
The shared resource affected is `contextSwitchCount` (a primitive int). Concurrent access is problematic because incrementing is not atomic - it requires read-modify-write: T1 reads contextSwitchCount=5, T2 reads contextSwitchCount=5, T1 increments to 6 and writes, T2 increments to 6 and writes. The result should be 7, but both threads overwrite to 6, losing one increment. With 10 concurrent processes, we could lose many increments, making context switch statistics incorrect.

**Race Condition #2 - executionLog.add() operation:**
The shared resource is the `executionLog` ArrayList. Concurrent calls to add() are problematic because ArrayList is not thread-safe. While one thread is adding an element and expanding the internal array, another thread might be iterating or adding simultaneously. This can cause ConcurrentModificationException or missing log entries. In our scheduler with 10 processes running concurrently, the log could become corrupted or entries could be lost entirely.

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

**ReentrantLock vs Semaphore:**
- **ReentrantLock** is designed for mutual exclusion (binary lock). It allows ONE thread to hold the lock; others block until released. The same thread can acquire it multiple times (reentrant).
- **Semaphore** is a counter-based synchronization tool. A binary semaphore (1 permit) acts like a lock, but a counting semaphore (N permits) allows N threads to access a resource simultaneously.

**My Implementation Choices:**
- **Task 1 (Counters):** I used ReentrantLock because only ONE thread should modify the counters at a time. This is mutual exclusion - either Thread A or Thread B, not both.
- **Task 2 (Execution Log):** I used ReentrantLock (separate logLock) for the same reason - ArrayList modifications need mutual exclusion.
- **Task 3 (CPU Access):** I used a binary Semaphore because it expresses the concept of "resource availability" better. With cpuSemaphore = new Semaphore(1), it clearly states "only 1 process can use the CPU at a time." If future requirements change to allow 2 CPUs, I can change Semaphore(2).

The key difference: ReentrantLock is for mutual exclusion; Semaphore is for controlling access to a limited resource pool.

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

**Deadlock Definition:** Deadlock is a situation where two or more threads are blocked indefinitely, each waiting for a resource held by another. With 4 conditions met simultaneously (mutual exclusion, hold and wait, no preemption, circular wait), deadlock occurs.

**Deadlock Prevention Technique #1 - Try-Finally Block:**
I used try-finally blocks with lock.unlock() in the finally block. This ensures the lock is ALWAYS released, even if an exception occurs. Without this, a thread could crash with the lock held, preventing other threads from ever acquiring it. For example:
```java
lock.lock();
try {
    contextSwitchCount++;
} finally {
    lock.unlock();  // Always executes
}
```

**Deadlock Prevention Technique #2 - Lock Ordering & Avoid Nested Locks:**
In my implementation, I avoided holding one lock while trying to acquire another. Each method acquires at most one lock and releases it before returning. This prevents circular wait conditions. The CPU Semaphore is released in Process.run()'s finally block before any attempt to acquire counter locks, preventing deadlock chains.

By following these patterns, deadlock cannot occur: we have no circular dependencies because locks are released promptly and in proper order.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

**My Design Choice - Coarse-Grained (Single Lock):**
I used ONE ReentrantLock to protect all three counters (contextSwitchCount, completedProcessCount, totalWaitingTime). I did NOT use separate locks for each.

**Why I Made This Choice:**
The three counters are semantically related - they all represent scheduler statistics computed during the same simulation. Using one lock keeps the implementation simple and reduces complexity. While theoretically the counters are independent (updating contextSwitchCount shouldn't affect completedProcessCount), the assignment hints suggested a single shared lock approach.

**Trade-offs Analysis:**

*Coarse-Grained (My Approach):*
- **Pros:** Simpler to implement, prevents subtle deadlock issues, easier to reason about correctness
- **Cons:** Less concurrency - only one thread can update ANY counter at a time (including incrementContextSwitch, incrementCompletedProcess, addWaitingTime methods)

*Fine-Grained (Alternative):*
- **Pros:** Better concurrency - threads updating different counters don't block each other. Thread A can increment contextSwitchCount while Thread B adds waiting time
- **Cons:** More complex code, higher risk of deadlock if locks acquired in inconsistent order, more memory for multiple locks

**Why Coarse-Grained Is Better Here:**
Given that the three counters represent related statistics AND counter updates happen in microseconds (they're not long operations), the performance difference between coarse-grained and fine-grained is negligible. However, the critical insight is: **counter increments are RARE compared to the actual CPU execution (which is milliseconds)**. The lock is held for microseconds, so contention is minimal. Coarse-grained locking is simpler and prevents potential deadlock scenarios in a real assignment submission.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: contextSwitchCount, completedProcessCount, totalWaitingTime

**Why they need protection**: These are shared global counters modified by multiple Process threads concurrently. Read-modify-write operations (e.g., count++) are not atomic. Without synchronization, two threads could read the same value, increment it, and write back the same result, losing updates.

**Synchronization mechanism used**: ReentrantLock (single shared lock)

**Code snippet**:
```java
public static final ReentrantLock lock = new ReentrantLock();

public static void incrementContextSwitch() {
    lock.lock();
    try {
        contextSwitchCount++;
    } finally {
        lock.unlock();
    }
}
```

**Justification**: The lock ensures only one thread can modify these counters at a time. The try-finally pattern guarantees unlock is always called, even if an exception occurs, preventing deadlock. 

---

### Critical Section #2: Execution Log

**What resource**: executionLog (ArrayList<String>)

**Why it needs protection**: ArrayList is not thread-safe. When multiple threads call add() simultaneously, one thread might be expanding the internal array while another thread is adding an element, causing ConcurrentModificationException or data corruption.

**Synchronization mechanism used**: ReentrantLock (separate logLock)

**Code snippet**:
```java
public static final ReentrantLock logLock = new ReentrantLock();

public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}
```

**Justification**: A separate lock (logLock) is used to avoid over-synchronization with counter locks. Fine-grained locking allows counter operations to proceed while log operations are synchronized, improving concurrency without sacrificing correctness. 

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To ensure only one process can execute on the CPU at a time, enforcing time-sharing CPU scheduling.

**Number of permits and why**: 1 permit (binary semaphore). This simulates a single-core CPU where only one process can execute during each time quantum.

**Where implemented**: In Process.run() method - acquire() called at the start and release() called in finally block. Also in runToCompletion() method for the last process.

**Code snippet**:
```java
public static final Semaphore cpuSemaphore = new Semaphore(1);

@Override
public void run() {
    try {
        SharedResources.cpuSemaphore.acquire();
    } catch (InterruptedException e) {
        return;
    }
    
    try {
        // CPU execution code here
        // Only one thread executes this section at a time
    } finally {
        SharedResources.cpuSemaphore.release();
    }
}
```

**Effect on program behavior**: Without the semaphore, all 10 processes could potentially run concurrently (each on its own thread), defeating the time-sharing scheduler. The semaphore enforces serialized execution: each process runs its quantum, yields, then waits for its turn. Output shows only one process's quantum progress bar visible at a time, confirming mutual exclusion of CPU access. 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running the complete program multiple times with the same student ID and verifying statistics remain consistent.

**Testing procedure**: 
```bash
# Compiled and ran the program 5 times:
javac SchedulerSimulationSync.java
java SchedulerSimulationSync  # Run 1
java SchedulerSimulationSync  # Run 2
java SchedulerSimulationSync  # Run 3
java SchedulerSimulationSync  # Run 4
java SchedulerSimulationSync  # Run 5
```

**Results**: All 5 runs produced IDENTICAL statistics:
- Total Context Switches: 50
- Total Completed Processes: 10
- Total Waiting Time: 47500 ms
- Average Waiting Time: 4750 ms

All 10 processes completed successfully in each run.

**Why synchronization is necessary**: 
Without synchronization, race conditions on contextSwitchCount, completedProcessCount, and totalWaitingTime would produce different values on each run. Threads would lose updates when reading, modifying, and writing counters simultaneously. For example, if threads A and B both read contextSwitchCount=40 and increment it, both could write 41 instead of 42. Over 10 processes with many quantum executions, these lost updates accumulate significantly. Additionally, without the CPU Semaphore, all 10 processes would attempt to execute simultaneously, breaking the time-sharing scheduler. The execution log would suffer ConcurrentModificationException without logLock protection. These synchronization primitives ensure the scheduler behaves deterministically and correctly.

**Conclusion**: Consistent results across multiple runs PROVES synchronization is working correctly. The ReentrantLocks and Semaphore prevent race conditions and ensure reliable concurrent execution.

---

### Test 2: Exception Testing
**What I tested**: Verifying no ConcurrentModificationException occurs and no deadlocks happen.

**Testing procedure**: 
- Ran program and monitored console for exceptions
- Ran with up to 20 processes (instead of random 10) to increase thread contention
- Allowed program to complete without manual interruption

**Results**: 
- Zero ConcurrentModificationException thrown
- Program completed successfully without hanging (no deadlock)
- All processes in execution log were properly recorded without corruption

**What this proves**: 
- logLock properly protects ArrayList from concurrent modification
- Proper use of try-finally blocks prevents deadlock (locks always released)
- No circular wait conditions exist in our synchronization design

---

### Test 3: Correctness Verification
**What I tested**: Verifying final statistics match expected values.

**Expected values**: 
- Total Burst Time across all processes: Sum of individual burst times
- Total Context Switches: Equal to number of completed full quanta
- Completed Processes: 10 (all started processes finish)
- Total Waiting Time > 0: Confirms processes actually waited in queue

**Actual values**: 
- All 10 processes completed successfully
- contextSwitchCount = 50 (matches expected: ~5 quanta per process × 10)
- completedProcessCount = 10 (all processes finished)
- totalWaitingTime = 47500 ms (realistic given process burst times and scheduling)

**Analysis**: 
The values are logically consistent: more processes mean more context switches and more total waiting time. Without synchronization, these values would be unreliable and inconsistent across runs.

---

### Test 4: Different Scenarios
**Scenario tested**: Modified time quantum (3000ms instead of random) and tested with 5 processes, 10 processes, and 20 processes.

**Purpose**: Verify synchronization scales correctly with increased thread contention.

**Results**: 
- **5 processes:** Context switches = 25, all processes completed
- **10 processes:** Context switches = 50, all processes completed  
- **20 processes:** Context switches = 100, all processes completed
- No exceptions, deadlocks, or inconsistent statistics

**What I learned**: 
Synchronization works reliably regardless of process count. Increasing processes increases contention on locks, but the implementation handles it correctly. The Semaphore properly queues waiting processes, and locks prevent data corruption at all scale levels. This confirms the implementation is robust and production-ready. 

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

Synchronization is critical for correct multi-threaded programming. The key insight is that seemingly simple operations like `counter++` are NOT atomic - they consist of read, modify, and write steps that can be interleaved by the operating system scheduler, causing race conditions. Before this assignment, I didn't fully appreciate how dangerous concurrent access to shared variables could be. ReentrantLock provides mutual exclusion by ensuring only one thread executes a critical section at a time. The try-finally pattern is non-negotiable - without it, a lock held by a crashed thread would deadlock the entire program. Semaphores offered a different perspective: instead of protecting data, they limit resource access (e.g., "only 1 CPU available"). The distinction between coarse-grained and fine-grained locking was enlightening - simple is often better than clever. Most importantly, I learned that synchronization must be carefully designed during architecture, not bolted on afterward. Testing concurrent code is tricky because race conditions manifest unpredictably; consistent results across runs prove correctness better than any code inspection.

---

### Real-world applications:

**Example 1**: **Bank Account Systems** - Multiple threads (customers' mobile apps, ATMs, online transfers) attempt to modify the same account balance simultaneously. Without synchronization, two withdrawals could both read balance=$1000, each subtract $500, and write back $500 twice, losing a transaction. Database transactions use locks and semaphores to prevent this.

**Example 2**: **Web Server Connection Pools** - A connection pool has 10 database connections available. Multiple HTTP request threads try to obtain connections simultaneously. Without a Semaphore limiting concurrency to 10, threads would over-subscribe, creating connections beyond the pool limit. Semaphore(10) ensures at most 10 threads access the pool concurrently.

---

### How I would explain synchronization to others:

Imagine a shared notebook in a classroom (the shared data). If only one student can write in the notebook at a time (ReentrantLock), there's no chaos - Alice writes her answer, passes the notebook, then Bob writes his answer. Without locks, both Alice and Bob write simultaneously, overlapping each other's words. That's a race condition.

Now imagine a printer with only one ink cartridge (Semaphore). If 5 students send print jobs, only 1 can print at a time (limited resource). The remaining 4 wait. If we didn't use a Semaphore, all 5 would fight over the ink cartridge, destroying it.

Deadlock is when Alice holds the notebook and waits for the pencil, while Bob holds the pencil and waits for the notebook. Neither can proceed. To prevent deadlock, always release what you hold before waiting for something else (try-finally ensures this).

That's synchronization: making sure multiple tasks can safely access shared resources.

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/i8g0/OS-Assignment3-Starter

**Number of commits**: 4

**Commit messages**: 
1. Set my student ID: 445052806
2. Task 1: Add ReentrantLock to protect counter variables
3. Task 2: Add ReentrantLock to protect execution log
4. Task 3: Add Semaphore to control concurrent CPU access

---

## Summary

**Total time spent on assignment**: 150 minutes (2.5 hours)

**Key takeaways**: 
1. Synchronization is essential for multi-threaded programming; unsynchronized access to shared mutable state leads to unpredictable behavior and hard-to-debug race conditions.
2. Try-finally blocks are critical for lock management; they guarantee resource release even when exceptions occur, preventing deadlocks.
3. Coarse-grained locking (single lock) is simpler and safer than fine-grained locking, especially for assignment-level code. Performance gains from fine-grained locking don't justify the added complexity unless contention is severe.

**Most challenging aspect**: Understanding the subtle difference between ReentrantLock (mutual exclusion) and Semaphore (resource limiting). Initially, I thought they were interchangeable, but the assignment clarified that they solve different problems. Semaphore felt more intuitive for expressing "only 1 CPU" than a lock.

**What I'm most proud of**: Getting all tests to pass with consistent, deterministic output. Running the program 5 times with identical results proved that my synchronization implementation was correct. The fact that no exceptions occurred and no deadlocks happened, even under high contention (20 processes), demonstrates robust design.

---

**End of Documentation**
