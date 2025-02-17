# LLD
Low Level Design

Resources:

### 1. [Watch the video on YouTube](https://www.youtube.com/watch?v=Fn0xBsmact4&list=PLvv0ScY6vfd_ocTP2ZLicgqKnvq50OCXM)
### 2. [Github.com](https://github.com/methylDragon/coding-notes/blob/master/C++)

### 3. [Medium.com Must Read](https://medium.com/@yc-kuo)
---

## SOLID Principles

### **Concurrency**
- **Definition**:  
  Concurrency refers to the ability of a system to manage multiple tasks (processes or threads) at the same time, giving the illusion of simultaneous execution. However, these tasks might not actually be executing at the same moment. Instead, the CPU switches between tasks rapidly (context switching).

- **Analogy**:  
  Think of a single chef preparing multiple dishes by switching between tasks like chopping vegetables, boiling water, and stirring a pot.

- **Example**:  
  If two threads share a single-core CPU, only one thread executes at any moment, but the OS schedules them in a way that they appear to run concurrently.

---

### **Parallelism**
- **Definition**:  
  Parallelism involves executing multiple tasks simultaneously on multiple processors or cores. Each task runs on its own processor without interference from others.

- **Analogy**:  
  Think of multiple chefs, each preparing their own dish simultaneously in a kitchen.

- **Example**:  
  On a multi-core CPU, two threads can run on separate cores at the same time, genuinely executing in parallel.

---

