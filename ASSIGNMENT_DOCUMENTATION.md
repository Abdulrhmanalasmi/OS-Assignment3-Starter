# Assignment 3 - Complete Documentation

**Student Name**: Abdulrhman Hamoud Alqahtani
**Student ID**: 443050700  
**Date Submitted**: May 2, 2026

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `443050700_Assignment3_Synchronization.mp4`

**Verification**:
- [x] Link is accessible (tested in incognito mode)
- [x] Video is 3-5 minutes long
- [x] Video shows code walkthrough and commits
- [x] Video has clear audio
- [x] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

### Entry 1 - May 2, 2026, 6:02 PM
**What I implemented**: Set my student ID to seed the random generator and initialized the assignment environment.
**Challenges encountered**: Ensuring the unique ID matches my actual university records to generate consistent random burst times.
**How I solved it**: Updated the `studentID` variable in the `main` method to `443050700`.
**Testing approach**: Ran the program to ensure the random processes generated were unique to my seed.
**Time spent**: 15 minutes.

---

### Entry 2 - May 2, 2026, 6:41 PM
**What I implemented**: Added fine-grained `ReentrantLock` instances to protect the shared counter variables (`contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`).
**Challenges encountered**: Choosing between a single coarse-grained lock or multiple fine-grained locks to avoid unnecessary performance bottlenecks.
**How I solved it**: Created three separate locks (`contextSwitchLock`, `completedProcessLock`, `waitingTimeLock`) since these variables are independent. Placed the `unlock()` methods inside `finally` blocks.
**Testing approach**: Ran the simulation multiple times to ensure the counters consistently matched the total number of processes.
**Time spent**: 45 minutes.

---

### Entry 3 - May 2, 2026, 7:17 PM
**What I implemented**: Implemented a `ReentrantLock` named `logLock` to protect the `executionLog` ArrayList from concurrent modifications.
**Challenges encountered**: Multiple threads writing to a non-thread-safe Java Collection simultaneously can corrupt the data structure.
**How I solved it**: Wrapped the `executionLog.add(message)` operation inside a `try-finally` block protected by `logLock`.
**Testing approach**: Checked the console output and the final log size to ensure no exceptions were thrown during execution.
**Time spent**: 30 minutes.

---

### Entry 4 - May 2, 2026, 7:49 PM
**What I implemented**: Added a binary `Semaphore` (`cpuSemaphore`) to restrict simulated CPU access to one process at a time.
**Challenges encountered**: Ensuring the semaphore is always released even if a thread is interrupted during execution.
**How I solved it**: Initialized `cpuSemaphore` with 1 permit. Called `acquire()` at the start of the `run()` and `runToCompletion()` methods, and `release()` inside the mandatory `finally` block.
**Testing approach**: Verified the console output to ensure quantum progress bars were printed sequentially without text overlapping from other threads.
**Time spent**: 45 minutes.

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:
1. **The Counter Race Condition**: The shared resource is `contextSwitchCount`. Concurrent access is a problem because the `++` operation is not atomic (it involves read, modify, and write steps). If two threads read the value simultaneously, increment it, and write it back, one increment is lost, resulting in an incorrect, lower count than expected.
2. **The Execution Log Race Condition**: The shared resource is the `executionLog` (ArrayList). Concurrent access is dangerous because standard Java ArrayLists are not thread-safe. If multiple threads call `.add()` at the exact same millisecond, the internal array structure can be corrupted, leading to lost entries or causing Java to throw a `ConcurrentModificationException`.

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:
A `ReentrantLock` provides mutual exclusion, ensuring that only the thread holding the lock can execute the critical section. It is owned by the acquiring thread. A `Semaphore` acts as a signaling mechanism controlling access to a shared resource pool based on a number of permits, and it doesn't have an "owner" thread. I used `ReentrantLock` to protect data structures in memory (the counters and the ArrayList) to prevent data corruption. I used a binary `Semaphore` to simulate the limited physical CPU resource, ensuring only one process thread "runs" on the processor at a time.

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:
Deadlock is a situation where two or more threads are blocked forever, each waiting for a lock held by the other. 
Two prevention techniques are: 
1. Always releasing locks in a `finally` block so an exception doesn't leave the lock permanently held. 
2. Acquiring multiple locks in a strict, consistent order to prevent circular waiting. 
In my code, I explicitly used the `try-finally` block for every single `ReentrantLock.lock()` and `Semaphore.acquire()` operation, guaranteeing that `unlock()` and `release()` are executed under all circumstances, completely eliminating deadlock risks.

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:

**Your Answer**:
I chose to use **fine-grained locking** by implementing three separate locks (`contextSwitchLock`, `completedProcessLock`, `waitingTimeLock`) rather than one coarse-grained lock for all three. I made this choice because updating `contextSwitchCount` is completely independent of updating `completedProcessCount` or `totalWaitingTime`. Using a single lock would force a thread updating the context switch counter to wait for a thread updating the completed process counter, creating an artificial and unnecessary bottleneck. The fine-grained approach provides significantly better concurrency, maximizing CPU utilization and parallelism, which represents industry best practices.

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`.
**Why they need protection**: They are modified by multiple threads concurrently, which can lead to lost updates due to non-atomic read-modify-write operations.
**Synchronization mechanism used**: Fine-Grained `ReentrantLock` (one for each counter).

**Code snippet**:
```java
public static final ReentrantLock contextSwitchLock = new ReentrantLock();

public static void incrementContextSwitch() {
    contextSwitchLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        contextSwitchLock.unlock();
    }
}
