# Linux Process Performance Debugging Guide

## Table of Contents
1. [Quick Reference](#quick-reference)
2. [Process Monitoring Fundamentals](#process-monitoring-fundamentals)
3. [CPU Performance Analysis](#cpu-performance-analysis)
4. [Memory Performance Analysis](#memory-performance-analysis)
5. [I/O Performance Analysis](#io-performance-analysis)
6. [Network Performance Analysis](#network-performance-analysis)
7. [Advanced Debugging Techniques](#advanced-debugging-techniques)
8. [Corner Cases and Gotchas](#corner-cases-and-gotchas)
9. [Performance Profiling Tools](#performance-profiling-tools)
10. [Troubleshooting Scenarios](#troubleshooting-scenarios)

## Quick Reference

### Essential Commands
```bash
# Process overview
ps aux --sort=-%cpu | head -20        # Top CPU consumers
ps aux --sort=-%mem | head -20        # Top memory consumers
top -p $(pgrep -d',' process_name)    # Monitor specific process

# Real-time monitoring
htop                                  # Interactive process viewer
iotop                                # I/O monitoring
nethogs                             # Network usage by process

# System load
uptime                              # Load averages
vmstat 1 5                         # Virtual memory stats
iostat -x 1 5                     # I/O statistics
```

## Process Monitoring Fundamentals

### Understanding Process States
```bash
# Process states in ps output
# R - Running
# S - Sleeping (interruptible)
# D - Sleeping (uninterruptible) - often I/O wait
# Z - Zombie
# T - Stopped
# I - Idle

# Check process states
ps axo pid,ppid,state,comm | grep -E '^[[:space:]]*[0-9]+.*D'  # Find D-state processes
```

### Process Hierarchy Analysis
```bash
# View process tree
pstree -p                          # Process tree with PIDs
pstree -u username                 # Process tree for specific user

# Parent-child relationships
ps -eo pid,ppid,comm,args --forest

# Find all children of a process
pgrep -P parent_pid

# Kill process and all children
pkill -TERM -P parent_pid
kill -TERM parent_pid
```

## CPU Performance Analysis

### CPU Utilization Monitoring
```bash
# Real-time CPU monitoring
top -d 1                           # 1-second updates
htop                              # More user-friendly

# Historical CPU data
sar -u 1 60                       # CPU utilization every second for 1 minute
mpstat 1 10                       # Multi-processor statistics

# Per-process CPU usage
pidstat -u 1                      # CPU usage per process
pidstat -u -p PID 1               # Specific process CPU usage
```

### CPU Affinity and Scheduling
```bash
# Check CPU affinity
taskset -cp PID                    # Show CPU affinity for process

# Set CPU affinity
taskset -cp 0,1 PID               # Bind process to CPUs 0 and 1
numactl --physcpubind=0-3 command # NUMA-aware CPU binding

# Process scheduling information
ps -eo pid,pri,ni,comm            # Priority and nice values
chrt -p PID                       # Real-time scheduling info

# Change process priority
renice -n 10 -p PID               # Lower priority (higher nice value)
chrt -f -p 99 PID                 # Set real-time FIFO priority
```

### Context Switching Analysis
```bash
# System-wide context switches
vmstat 1 | awk '{print $12}'      # Context switches per second

# Per-process context switches
grep voluntary_ctxt_switches /proc/PID/status
grep nonvoluntary_ctxt_switches /proc/PID/status

# Monitor context switches over time
pidstat -w 1                      # Context switch statistics
```

## Memory Performance Analysis

### Memory Usage Monitoring
```bash
# System memory overview
free -h                           # Human-readable memory info
cat /proc/meminfo                 # Detailed memory statistics

# Per-process memory usage
ps aux --sort=-%mem               # Sort by memory usage
pmap -x PID                       # Detailed memory map
cat /proc/PID/smaps               # Detailed memory segments

# Memory usage over time
pidstat -r 1                      # Memory statistics per process
sar -r 1 60                       # System memory usage
```

### Memory Leak Detection
```bash
# Track memory growth
while true; do
    echo "$(date): $(ps -o rss= -p PID)" >> memory_usage.log
    sleep 60
done

# Valgrind for detailed analysis
valgrind --tool=memcheck --leak-check=full ./your_program

# Using massif for heap profiling
valgrind --tool=massif ./your_program
ms_print massif.out.PID
```

### Virtual Memory Analysis
```bash
# Virtual memory statistics
vmstat 1 5                        # Page in/out, swap usage

# Page fault analysis
pidstat -r -p PID 1               # Page faults per process

# Swap usage per process
for pid in $(pgrep process_name); do
    awk '/VmSwap/{print FILENAME":"$2}' /proc/$pid/status
done
```

## I/O Performance Analysis

### Disk I/O Monitoring
```bash
# System I/O overview
iostat -x 1 5                     # Extended I/O statistics
iotop -o                          # Only show processes doing I/O

# Per-process I/O
pidstat -d 1                      # Disk I/O per process
cat /proc/PID/io                  # Detailed I/O counters

# I/O wait analysis
top -d 1                          # Look for high %wa (I/O wait)
vmstat 1 | awk '{print $16}'     # I/O wait time percentage
```

### File and Directory Access
```bash
# Monitor file access
strace -e trace=file -p PID       # File system calls
lsof -p PID                       # Open files by process
lsof +D /path/to/directory        # Processes accessing directory

# Find large files being accessed
lsof | awk '$7 ~ /[0-9]+/ && $7 > 1000000 {print $0}'

# Monitor file descriptor usage
ls /proc/PID/fd | wc -l           # Count open file descriptors
ulimit -n                        # Max file descriptors per process
```

### Block Device Performance
```bash
# Identify I/O bottlenecks
iotop -a                          # Accumulated I/O
blktrace /dev/sda                 # Detailed block I/O tracing

# Check device queue depth
cat /sys/block/sda/queue/nr_requests

# Monitor specific device
iostat -x sda 1                   # Device-specific I/O stats
```

## Network Performance Analysis

### Network Connection Monitoring
```bash
# Network connections per process
netstat -tulpn | grep PID
ss -tulpn | grep PID              # Modern alternative to netstat

# Network usage per process
nethogs                           # Real-time network usage
iftop                            # Interface-level network usage

# Monitor specific connections
tcpdump -i eth0 host 192.168.1.100
```

### Socket Analysis
```bash
# Socket statistics
ss -s                             # Socket summary
ss -tuln                         # TCP/UDP listening sockets

# Per-process socket info
lsof -i -a -p PID                # Network files for specific process
netstat -p | grep PID            # Network connections for process
```

## Advanced Debugging Techniques

### System Call Tracing
```bash
# Trace system calls
strace -p PID                     # Attach to running process
strace -c -p PID                  # Count system calls
strace -T -p PID                  # Time each system call
strace -f -p PID                  # Follow child processes

# Filter specific system calls
strace -e trace=read,write -p PID
strace -e trace=network -p PID    # Network-related calls
strace -e trace=file -p PID       # File-related calls
```

### Profiling with perf
```bash
# CPU profiling
perf top                          # Real-time CPU profiling
perf record -p PID sleep 30       # Record for 30 seconds
perf report                       # Analyze recorded data

# Cache performance
perf stat -p PID                  # Performance counters
perf stat -e cache-misses -p PID  # Cache miss analysis

# Function-level profiling
perf record -g -p PID             # Call graph recording
```

### Process Resource Limits
```bash
# Check current limits
cat /proc/PID/limits              # All limits for process
ulimit -a                        # Current shell limits

# Common limits to check
ulimit -n                        # File descriptors
ulimit -u                        # Max user processes
ulimit -v                        # Virtual memory
ulimit -s                        # Stack size

# System-wide limits
cat /proc/sys/fs/file-max        # System file descriptor limit
sysctl kernel.pid_max            # Maximum PID value
```

## Corner Cases and Gotchas

### Zombie Processes
```bash
# Identify zombie processes
ps aux | awk '$8 ~ /^Z/ { print $2 }'

# Find parent of zombie
ps -eo pid,ppid,state,comm | grep Z

# Force parent to reap zombies (last resort)
kill -CHLD parent_pid
```

**Corner Case**: Zombie processes don't consume resources except PID table entry, but too many can exhaust PID space.

### D-State Processes (Uninterruptible Sleep)
```bash
# Find D-state processes
ps axo pid,ppid,state,comm | grep ' D '

# Check what they're waiting for
cat /proc/PID/wchan               # Wait channel
cat /proc/PID/stack               # Kernel stack trace
```

**Corner Case**: D-state processes cannot be killed with SIGKILL. Usually indicates I/O issues or kernel bugs.

### High Load but Low CPU Usage
```bash
# Check for I/O wait
vmstat 1                          # Look for high 'wa' column
iostat -x 1                       # Check for I/O bottlenecks

# Check for excessive context switching
vmstat 1 | awk '{print $12}'     # Context switches per second

# Look for lock contention
perf record -e sched:sched_stat_blocked -ag sleep 10
```

**Corner Case**: High load average with low CPU utilization often indicates I/O wait or lock contention.

### Memory Fragmentation Issues
```bash
# Check memory fragmentation
cat /proc/buddyinfo               # Buddy allocator info
cat /proc/pagetypeinfo            # Page type information

# Check for memory compaction
grep compact /proc/vmstat
```

**Corner Case**: Available memory exists but large allocations fail due to fragmentation.

### NUMA-related Performance Issues
```bash
# Check NUMA topology
numactl --hardware
lscpu | grep NUMA

# NUMA memory usage per node
numastat
numastat -p PID                   # Per-process NUMA stats

# Check for cross-NUMA access
perf stat -e node-loads,node-load-misses -p PID
```

**Corner Case**: Process accessing memory from remote NUMA nodes can cause significant performance degradation.

### Process Group and Session Issues
```bash
# Check process group and session
ps -eo pid,pgid,sid,comm

# Orphaned process groups
ps -eo pid,pgid,ppid,comm | awk '$3 == 1 && $1 != $2'
```

**Corner Case**: Orphaned processes may not receive expected signals or may hang on terminal I/O.

### File Descriptor Leaks
```bash
# Monitor file descriptor usage
watch -n 1 'ls /proc/PID/fd | wc -l'

# Find processes with most open files
lsof | awk '{print $2}' | sort | uniq -c | sort -nr | head -10

# Check for specific file types
lsof -p PID | grep -E '(REG|socket|pipe)'
```

**Corner Case**: File descriptor leaks can cause "Too many open files" errors even when system limits seem adequate.

### CPU Throttling Issues
```bash
# Check CPU throttling
grep . /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq
cat /proc/cpuinfo | grep MHz

# Thermal throttling
sensors                           # Temperature monitoring
dmesg | grep -i thermal
```

**Corner Case**: CPU throttling due to thermal limits can cause inconsistent performance.

## Performance Profiling Tools

### perf - Linux Performance Analysis
```bash
# Basic profiling
perf top                          # Real-time profiling
perf top -p PID                   # Profile specific process

# Record and analyze
perf record -g ./program          # Record with call graphs
perf record -p PID sleep 30       # Record running process
perf report --stdio               # Text output
perf report --tui                 # Interactive TUI

# Event-specific profiling
perf list                         # Available events
perf stat -e cache-misses,cache-references ./program
perf record -e cpu-clock -g -p PID

# Memory profiling
perf record -e page-faults -g -p PID
perf mem record ./program         # Memory access profiling
perf mem report
```

### Flamegraphs
```bash
# Generate flamegraph
perf record -F 99 -g -p PID sleep 30
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg

# Off-CPU flamegraph
perf record -e sched:sched_stat_sleep -e sched:sched_switch \
    -e sched:sched_stat_iowait -g -p PID sleep 30
```

### Advanced Memory Analysis
```bash
# Address Sanitizer (compile-time)
gcc -fsanitize=address -g program.c

# Valgrind tools
valgrind --tool=memcheck --track-origins=yes ./program
valgrind --tool=callgrind ./program
valgrind --tool=massif ./program

# Memory usage tracking
/usr/bin/time -v ./program        # Detailed memory stats
```

## Troubleshooting Scenarios

### Scenario 1: High CPU Usage but Low Throughput
```bash
# Steps to diagnose:
1. Check for excessive context switching
   vmstat 1 | awk '{print $12}'

2. Look for lock contention
   perf record -e sched:sched_stat_blocked -p PID sleep 10

3. Analyze system calls
   strace -c -p PID

4. Check CPU scheduling
   ps -eo pid,pri,ni,psr,comm | grep PID
```

### Scenario 2: Memory Usage Keeps Growing
```bash
# Steps to diagnose:
1. Monitor memory growth
   watch -n 5 'ps -o rss= -p PID'

2. Check for memory leaks
   valgrind --tool=memcheck --leak-check=full ./program

3. Analyze heap usage
   pmap -x PID

4. Check for mmap leaks
   cat /proc/PID/maps | wc -l
```

### Scenario 3: Process Hangs or Becomes Unresponsive
```bash
# Steps to diagnose:
1. Check process state
   ps -o pid,state,wchan,comm | grep PID

2. Get stack trace
   gdb -p PID
   (gdb) bt
   (gdb) info threads

3. Check system calls
   strace -p PID

4. Look for deadlocks
   pstack PID  # If available
```

### Scenario 4: I/O Performance Issues
```bash
# Steps to diagnose:
1. Check I/O wait
   iostat -x 1 5

2. Monitor process I/O
   pidstat -d -p PID 1

3. Check open files
   lsof -p PID

4. Analyze I/O patterns
   strace -e trace=read,write,lseek -p PID
```

### Scenario 5: Network Performance Problems
```bash
# Steps to diagnose:
1. Monitor network usage
   nethogs
   iftop -i interface

2. Check connection states
   ss -tuln | grep :port

3. Analyze network calls
   strace -e trace=network -p PID

4. Capture packets
   tcpdump -i interface -w capture.pcap
```

### Emergency Debugging
```bash
# Quick health check script
#!/bin/bash
echo "=== System Load ==="
uptime

echo "=== Memory Usage ==="
free -h

echo "=== Top CPU Consumers ==="
ps aux --sort=-%cpu | head -5

echo "=== Top Memory Consumers ==="
ps aux --sort=-%mem | head -5

echo "=== Disk Usage ==="
df -h

echo "=== I/O Statistics ==="
iostat -x 1 1
```

## Useful One-liners

```bash
# Find processes using the most CPU
ps aux | sort -nrk 3,3 | head -5

# Find processes using the most memory
ps aux | sort -nrk 4,4 | head -5

# Monitor process memory usage over time
watch -n 1 "ps -o pid,rss,vsz,comm -p PID"

# Find which process is using a specific port
lsof -i :port_number

# Kill all processes by name
pkill process_name

# Find large files being accessed
lsof | awk '$7 ~ /[0-9]+/ && $7 > 100000000'

# Check if process is swapping
grep VmSwap /proc/PID/status

# Monitor system call frequency
strace -c -p PID 2>&1 | tail -n +3 | head -n -2 | sort -nrk 1,1

# Find processes in D-state
ps axo pid,ppid,state,comm | awk '$3=="D"'
```

---
