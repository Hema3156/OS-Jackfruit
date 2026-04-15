# OS Jackfruit — Lightweight Container Runtime

---

## 👥 Team Information

* HEEMADHAWALA R — PES2UG24CS190
* HEMAVATHI E — PES2UG24CS193

---

## ⚙️ Build, Load, and Run Instructions

### 🔹 Build

make clean
make

---

### 🔹 Load Kernel Module

sudo insmod monitor.ko

Verify:
ls /dev/container_monitor

---

### 🔹 Start Supervisor

sudo ./engine supervisor rootfs-base

---

### 🔹 Start Containers

sudo ./engine start alpha rootfs-base /bin/sh
sudo ./engine start beta rootfs-base /bin/sh

---

### 🔹 List Containers

sudo ./engine ps

---

### 🔹 View Logs

sudo ./engine logs <id>

---

### 🔹 Memory Test

cp memory_hog rootfs-base/
sudo ./engine start soft rootfs-base ./memory_hog
sudo dmesg | grep container_monitor

---

### 🔹 Stop Container

sudo ./engine stop <id>

---

### 🔹 Cleanup

sudo pkill engine
sudo rmmod monitor

---

## 📸 Demo with Screenshots

---

### 📸 Screenshot 1 — Multi-container supervision

Two containers (alpha, beta) running simultaneously under a single supervisor process.
<img width="800" alt="Multi-container supervision" src="https://github.com/user-attachments/assets/146b79ec-afe4-4e10-b567-5bc2c7482df5" />

---

### 📸 Screenshot 2 — Metadata tracking

Output of `engine ps` showing container ID, PID, state, and memory limits.
<img width="1280" height="832" alt="Metadata tracking" src="https://github.com/user-attachments/assets/ae0061e3-7c4e-4b6f-a3ef-12746daf35fc" />


---

### 📸 Screenshot 3 — Bounded-buffer logging

Log output captured through the producer–consumer logging pipeline.
<img width="1280" height="832" alt="Bounded-buffer logging" src="https://github.com/user-attachments/assets/007da410-f431-4702-b79a-70e36ebb2eb3" />

---

### 📸 Screenshot 4 — CLI and IPC

A CLI `stop` command sent to the supervisor via UNIX domain socket and handled correctly.
<img width="1280" height="832" alt=" CLI and IPC" src="https://github.com/user-attachments/assets/911d3515-51e0-4e72-8565-a15a74f5d84d" />

---

### 📸 Screenshot 5 — Soft-limit warning

Kernel logs showing warning when container exceeds soft memory limit.

<img width="1600" height="548" alt="image" src="https://github.com/user-attachments/assets/28198432-5279-4b84-be52-11fcac27bcf4" />

<img width="1514" height="296" alt="Soft-limit warning" src="https://github.com/user-attachments/assets/8bcb18c8-afde-4588-851e-73877920e9eb" />

---

### 📸 Screenshot 6 — Hard-limit enforcement

Kernel logs showing process termination when container exceeds hard memory limit.

<img width="1600" height="548" alt="image" src="https://github.com/user-attachments/assets/28198432-5279-4b84-be52-11fcac27bcf4" />

<img width="1490" height="208" alt="Hard-limit enforcement" src="https://github.com/user-attachments/assets/8cd38630-a3d0-4c7c-abe8-2be9fc6aa9ba" />

---

### 📸 Screenshot 7 — Scheduling experiment

Two containers running CPU workloads with different nice values showing scheduling behavior.
<img width="1256" height="520" alt="image" src="https://github.com/user-attachments/assets/0d7cf825-e204-44ef-ad4e-3ba4fbe32fff" />

<img width="1280" height="832" alt="Scheduling experiment" src="https://github.com/user-attachments/assets/9e199736-e5d4-4f71-9434-7ec45d32218f" />



---

### 📸 Screenshot 8 — Clean teardown

All processes stopped and kernel module unloaded cleanly.

<img width="1564" height="456" alt="Clean teardown" src="https://github.com/user-attachments/assets/8ecc8606-373f-418e-b850-6bfbd4381a5f" />

---

## ⚙️ Engineering Analysis

### 🔹 Isolation Mechanisms

Containers use Linux namespaces:

* PID namespace for separate process IDs
* UTS namespace for hostname isolation
* Mount namespace for filesystem isolation

Each container uses `chroot()` to isolate its root filesystem.

---

### 🔹 Supervisor and Process Lifecycle

A long-running supervisor manages all containers.
It:

* creates containers using `clone()`
* tracks metadata (PID, state, limits)
* handles `SIGCHLD` to reap processes
* updates container states

---

### 🔹 IPC, Threads, and Synchronization

Two IPC mechanisms are used:

1. UNIX Domain Socket

   * Used between CLI and supervisor

2. Pipes + Threads

   * Used for logging (stdout/stderr)

Synchronization:

* Mutex + condition variables for bounded buffer
* Prevents race conditions and data loss

---

### 🔹 Memory Management and Enforcement

A kernel module monitors memory using RSS:

* Soft limit → logs warning
* Hard limit → sends SIGKILL

Implemented using:

* Linked list of monitored processes
* Timer-based periodic checking
* ioctl interface between user and kernel

---

### 🔹 Scheduling Behavior

Scheduling is tested using different nice values:

* Lower nice value → higher priority
* Higher nice value → lower priority

Observation:
Processes with higher priority execute faster.

---

## 🧩 Design Decisions and Tradeoffs

* clone() → lightweight container creation
* chroot() → simple filesystem isolation
* UNIX socket → efficient IPC
* bounded buffer → safe concurrent logging
* kernel module → accurate memory tracking

Tradeoff:

* chroot is simpler but less secure than pivot_root

---

## 📊 Scheduler Experiment Results

| Process | Nice Value | Behavior         |
| ------- | ---------- | ---------------- |
| cpu_hog | -5         | Faster execution |
| cpu_hog | 10         | Slower execution |

---

## ✅ Conclusion

This project demonstrates:

* Container creation using namespaces
* IPC using UNIX sockets
* Bounded-buffer logging
* Kernel-level memory monitoring
* Scheduling behavior
* Proper cleanup with no zombie processes

---

## 📌 Learning Outcomes

* Linux namespaces
* Kernel module development
* Process scheduling
* IPC mechanisms
* System programming

---
