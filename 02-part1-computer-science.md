# Part 1: Computer Science Foundations

---

**Objective:** Build an unshakeable foundation in how computers work, how data is represented, how operating systems function, and how networks communicate.

**Duration:** 8 weeks (Chapters 1-8)

---

# Chapter 1: Computer Fundamentals

## Introduction

A computer is an electronic device that processes data according to a set of instructions. Before you write your first line of code, you must understand what happens inside the machine that runs your code. This chapter covers the fundamental building blocks of all modern computing systems.

## Why It Matters

Every line of code you write ultimately becomes electrical signals moving through silicon. Understanding how computers work helps you write efficient code, debug complex issues, and design better systems. Senior engineers at Google, Microsoft, and Amazon all have deep understanding of computer fundamentals.

## Real World Analogy

Think of a computer as a kitchen:

| Component | Kitchen Analogy |
|-----------|----------------|
| CPU | The Chef (does the actual work) |
| RAM | The Countertop (workspace for active tasks) |
| SSD/HDD | The Pantry (storage for ingredients) |
| Motherboard | The Kitchen Layout (connects everything) |
| Operating System | The Recipe Book Manager (organizes tasks) |

## Theory

### What is a Computer?

A computer is a programmable machine that:
1. **Accepts input** (data)
2. **Processes data** (computation)
3. **Stores data** (memory)
4. **Produces output** (results)

This is the fundamental **Input → Process → Storage → Output** model.

### The Von Neumann Architecture

Modern computers are based on the Von Neumann architecture, proposed by John von Neumann in 1945:

```
┌──────────────────────────────────────────────────┐
│                    CPU                            │
│  ┌──────────┐    ┌───────────────────────────┐   │
│  │   CU     │◄──►│         ALU               │   │
│  │ Control  │    │  Arithmetic Logic Unit    │   │
│  │  Unit    │    └───────────────────────────┘   │
│  └────┬─────┘                                     │
│       │                                           │
│  ┌────▼──────────────────────────────────────┐    │
│  │              Registers                     │    │
│  └───────────────────────────────────────────┘    │
└──────────────────────┬───────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────┐
│                  Memory (RAM)                     │
│  ┌──────┬──────┬──────┬──────┬──────┬──────┐     │
│  │ Inst │ Inst │ Data │ Data │ Data │ Inst │     │
│  │  1   │  2   │  A   │  B   │  C   │  3   │     │
│  └──────┴──────┴──────┴──────┴──────┴──────┘     │
└──────────────────────────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────┐
│           Storage (SSD/HDD)                       │
└──────────────────────────────────────────────────┘
```

**Key Components:**

1. **CPU (Central Processing Unit):** The brain. Executes instructions.
   - **CU (Control Unit):** Directs operations, fetches instructions
   - **ALU (Arithmetic Logic Unit):** Performs calculations and logical operations
   - **Registers:** Ultra-fast, tiny memory inside the CPU (typically 32-64 per core)

2. **Memory (RAM):** Temporary storage for active programs. Volatile (lost on power off).

3. **Storage (SSD/HDD):** Permanent storage. Non-volatile.

4. **Motherboard:** Connects all components via buses (data pathways).

5. **Input/Output Devices:** Keyboard, mouse, monitor, network card, etc.

### Memory Hierarchy

```
Register:        ~1 cycle       ~1 KB total
L1 Cache:        ~3 cycles      ~32 KB per core
L2 Cache:        ~10 cycles     ~256 KB per core
L3 Cache:        ~40 cycles     ~8-32 MB shared
RAM:             ~200 cycles    ~8-64 GB
SSD:             ~100k cycles   ~256 GB - 2 TB
HDD:             ~10M cycles    ~1-10 TB
```

Each level is larger, slower, and cheaper than the level above it.

> 💡 **Pro Tip:** Writing cache-friendly code (sequential memory access patterns) can improve performance by 10-100x compared to random access patterns. Always consider the memory hierarchy when designing data structures.

## Internal Working: The Fetch-Decode-Execute Cycle

```
Step 1: FETCH
  Program Counter → Memory → Instruction Register

Step 2: DECODE
  Control Unit interprets the instruction

Step 3: EXECUTE
  ALU performs operation, or data is moved

Step 4: UPDATE PC
  Program Counter advances to next instruction
```

This cycle happens billions of times per second (3-5 GHz in modern CPUs).

> 💡 **Pro Tip:** Understanding the fetch-decode-execute cycle helps you reason about performance. Each CPU instruction takes a few cycles, so minimizing the number of instructions executed in hot paths is key to optimization.

## Common Mistakes

1. **Confusing RAM with Storage:** RAM is volatile; storage is persistent.
2. **Thinking more GHz always means faster:** Architecture matters more than clock speed.
3. **Ignoring the memory hierarchy:** Cache misses can cost 100x performance.

## Best Practices

- Know your hardware: CPU cores, RAM size, disk type.
- Monitor resource usage: Task Manager, `htop`, Activity Monitor.
- Consider the full stack: optimization at any layer affects the whole system.

## Interview Questions

1. **Q:** What is the difference between RAM and ROM?  
   **A:** RAM (Random Access Memory) is volatile, used for temporary storage. ROM (Read Only Memory) is non-volatile, stores firmware permanently.

2. **Q:** Explain the Fetch-Decode-Execute cycle.  
   **A:** The CPU fetches an instruction from memory at the address in the Program Counter. It decodes the instruction in the Control Unit. It executes using the ALU. Finally, it updates the Program Counter.

3. **Q:** What is cache memory and why is it important?  
   **A:** Cache is ultra-fast memory inside the CPU. It stores frequently accessed data to reduce fetch time from main RAM. Modern CPUs have L1, L2, and L3 cache levels.

## Practical Exercises

1. Open Task Manager (Windows) or Activity Monitor (Mac). Identify your CPU, RAM, and disk usage.
2. Run a program and observe how RAM usage changes.
3. Research your computer's CPU specs: cores, threads, clock speed, cache size.

## Revision Notes

- Computer = Input → Process → Storage → Output
- Von Neumann: CPU (CU + ALU + Registers) + Memory + Storage
- Memory hierarchy: Register > Cache > RAM > SSD > HDD
- Fetch-Decode-Execute cycle runs billions of times/second

---

# Chapter 2: Binary, Decimal, and Hexadecimal

## Introduction

Computers understand only two things: 0 and 1. Every piece of data — numbers, text, images, videos — is ultimately represented as sequences of binary digits. This chapter teaches number systems.

## Why It Matters

- **IP addresses** in decimal, processed in binary
- **Memory addresses** shown in hexadecimal
- **Color codes** in CSS: `#FF0000` = red
- **Bitwise operations** in performance-critical code
- **Cryptography** operates on binary data

## Real World Analogy

Different number systems are like different languages saying the same number:
- **Decimal (Base 10):** "42" = 4 tens + 2 ones
- **Binary (Base 2):** "101010" = 32 + 0 + 8 + 0 + 2 + 0 = 42
- **Hexadecimal (Base 16):** "2A" = 2×16 + 10 = 42

## Theory

### Number Systems

| System | Base | Digits | Example |
|--------|------|--------|---------|
| Binary | 2 | 0, 1 | 1010₂ |
| Decimal | 10 | 0-9 | 10₁₀ |
| Hexadecimal | 16 | 0-9, A-F | A₁₆ |
| Octal | 8 | 0-7 | 12₈ |

### Positional Notation

Each position represents a power of the base:

```
Decimal 4,206:
  4 × 10³ + 2 × 10² + 0 × 10¹ + 6 × 10⁰ = 4,206

Binary 1101:
  1 × 2³ + 1 × 2² + 0 × 2¹ + 1 × 2⁰ = 13

Hex 2F:
  2 × 16¹ + 15 × 16⁰ = 47
```

## Internal Working: Number Storage

### Unsigned Integers

8-bit unsigned: 0 to 255

```
Bits:    7   6   5   4   3   2   1   0
Value: 128  64  32  16   8   4   2   1

173 = 1 0 1 0 1 1 0 1 = 128+32+8+4+1
```

### Signed Integers (Two's Complement)

Leftmost bit = sign (0 positive, 1 negative)

```
+42 = 00101010
-42 = 11010110 (invert all bits, add 1)
```

### IEEE 754 Floating Point

```
32-bit float:
┌─┬────────┬─────────────────────────────┐
│1│   8    │             23              │
│S│Exponent│          Mantissa           │
└─┴────────┴─────────────────────────────┘
```

> ⚠️ **Warning:** Floating-point arithmetic is not exact. Never compare floats with `==`. Use an epsilon threshold: `Math.abs(a - b) < 1e-9` instead. This is critical in financial or scientific computing.

## Conversions

### Binary ↔ Decimal

```
101101₂ → 32 + 0 + 8 + 4 + 0 + 1 = 45₁₀

45₁₀ → 45÷2=22r1, 22÷2=11r0, 11÷2=5r1, 5÷2=2r1, 2÷2=1r0, 1÷2=0r1
      → Read bottom to top: 101101₂
```

### Binary ↔ Hex

Group in sets of 4 (right to left):

```
101101₂ → 0010 1101 → 2D₁₆
```

> 💡 **Pro Tip:** When debugging, use hex to inspect raw memory — it maps directly to binary (1 hex digit = 4 binary bits) and is far more readable than long binary strings. Most debugger memory views default to hex.

## Common Mistakes

1. **Off-by-one in bit positions:** Bits are 0-indexed from the right.
2. **Confusing binary and decimal:** `10` in binary = 2 in decimal.
3. **Integer overflow:** When a number exceeds its representable range.

## Best Practices

- Use hex for debugging (memory dumps, network packets).
- Know your integer sizes: `int` = 32 bits, `long` = 64 bits in Java.
- Watch for overflow: use `long` or `BigInteger` for large numbers.

## Interview Questions

1. **Q:** Convert 255 from decimal to binary and hex.  
   **A:** Binary: 11111111, Hex: FF.

2. **Q:** What is two's complement and why is it used?  
   **A:** Method for representing signed integers. Advantages: single representation for zero, simple addition/subtraction hardware.

3. **Q:** Range of a 32-bit signed integer?  
   **A:** -2,147,483,648 to 2,147,483,647.

## Practical Exercises

1. Convert 127, 256, 1024 to binary and hex.
2. Add 101101 + 11011 in binary.
3. Represent -42 in 8-bit two's complement.

## Mini Project: Number System Converter

Build a CLI tool that converts between binary, decimal, hex, and octal.

## Revision Notes

- Binary = Base 2 (0, 1)
- Hex = Base 16 (0-9, A-F)
- Each hex digit = 4 binary digits
- Two's complement: invert + 1
- IEEE 754: 1 sign, 8 exponent, 23 mantissa bits (float)

---

# Chapter 3: Data Representation

## Introduction

All data in a computer is binary. We need conventions (encodings) to interpret binary as meaningful information. This chapter covers text, image, and audio encoding.

## Why It Matters

- Character encoding bugs cause corrupted text (mojibake)
- Image formats affect quality and file size
- Endianness matters when transferring data between systems

## Real World Analogy

Data representation is like a codebook:
- Binary `01000001` as ASCII = 'A'
- Binary `01000001` as integer = 65
- Same bits, different meaning

## Text Encoding

### ASCII (7-bit, 128 characters)

```
A = 65 = 01000001
a = 97 = 01100001
0 = 48 = 00110000
```

### Unicode and UTF-8

| Feature | ASCII | UTF-8 |
|---------|-------|-------|
| Bits per char | 7 | 8-32 (variable) |
| Characters | 128 | 1M+ |
| Backward compatible | — | Yes (ASCII = valid UTF-8) |
| Web usage | <1% | 98%+ |

```
A  U+0041    01000001
€  U+20AC    11100010 10000010 10101100
🌀 U+1F300   11110000 10011111 10001100 10000000
```

### UTF-8 Encoding Rules

| Bytes | Bits | Pattern |
|-------|------|---------|
| 1 | 7 | 0xxxxxxx |
| 2 | 11 | 110xxxxx 10xxxxxx |
| 3 | 16 | 1110xxxx 10xxxxxx 10xxxxxx |
| 4 | 21 | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |

> ⚠️ **Warning:** Always specify the character encoding (`<meta charset="UTF-8">`, `encoding="utf-8"`) in your projects. Mismatched encodings cause mojibake (garbled text) that can take hours to diagnose.

## Image Representation

### Bitmap Images

Each pixel has color values:

```
RGB Color Model: 3 bytes/pixel (Red, Green, Blue)
  24-bit color: 16.7 million colors
  32-bit: adds Alpha (transparency)
```

### Image Formats

| Format | Compression | Quality | Use Case |
|--------|-------------|---------|----------|
| PNG | Lossless | High | Web graphics, screenshots |
| JPEG | Lossy | Adjustable | Photographs |
| WebP | Both | High | Modern web |
| SVG | Vector | Infinite | Icons, logos |

## Audio Representation

Digital audio samples sound waves at regular intervals:

- **Sample Rate:** Number of samples/second (Hz). CD quality = 44,100 Hz
- **Bit Depth:** Bits per sample (16-bit, 24-bit)
- **Channels:** Mono (1), Stereo (2), 5.1 (6)

**Calculation:** Stereo, 44.1 kHz, 16-bit = 44,100 × 2 × 2 = 176,400 bytes/sec ≈ 10.6 MB/min

## Endianness

Byte order for multi-byte values:

```
32-bit integer 0x12345678:

Little Endian (x86):        78 56 34 12
Big Endian (network order): 12 34 56 78
```

> ❓ **Interview Question:** What happens when a little-endian system receives data from a big-endian system over the network? — Network protocols use big endian (network byte order). The receiver must convert using `ntohl`/`htonl` functions to avoid byte-swapped values.

## Common Mistakes

1. **Assuming ASCII:** Most systems use UTF-8.
2. **Confusing endianness:** Network byte order is big endian.
3. **Lossy compression for wrong use case:** JPEG for text screenshots.

## Interview Questions

1. **Q:** Difference between UTF-8 and UTF-16?  
   **A:** UTF-8: 1-4 bytes/char, ASCII-compatible. UTF-16: 2 or 4 bytes/char, more efficient for non-ASCII.

2. **Q:** How is color represented in digital images?  
   **A:** RGB model, each component 0-255 (8 bits), 16.7M colors.

## Revision Notes

- ASCII: 7-bit, 128 chars
- UTF-8: variable 1-4 bytes, backward compatible with ASCII
- PNG = lossless, JPEG = lossy
- Sample rate × bit depth × channels = audio bitrate
- Little endian: LSB first (x86). Big endian: MSB first (network)

---

# Chapter 4: CPU Architecture

## Introduction

The CPU is the heart of the computer. This chapter covers CPU architecture: pipelining, caching, multi-core processing, and instruction-level parallelism.

## Why It Matters

- Algorithm efficiency depends on CPU architecture
- Multi-threading requires understanding cores and caches
- Cache-unfriendly algorithms tank performance

## Real World Analogy

A CPU is like a factory assembly line:
- **Pipeline = Assembly line** (multiple stages simultaneously)
- **Execution Units = Workers** (ALU, FPU, etc.)
- **Registers = Tool carts** (immediate access)
- **Cache = Storage room** (fast access)
- **RAM = Warehouse** (slower, bulk)

## CPU Core Components

```
┌──────────────────────────────────────────┐
│              CPU Core                     │
│  ┌──────┐  ┌──────┐  ┌──────┐          │
│  │ L1   │  │ L1   │  │ L2  │          │
│  │ Inst │  │ Data │  │Cache│          │
│  └──┬───┘  └──┬───┘  └──┬───┘          │
│     │         │         │               │
│  ┌──┴─────────┴─────────┴───┐           │
│  │     Fetch / Decode       │           │
│  └──────────┬───────────────┘           │
│             │                            │
│  ┌──────────▼───────────────┐           │
│  │    Execution Engine      │           │
│  │  ┌────┐ ┌────┐ ┌────┐  │           │
│  │  │ALU │ │FPU │ │SIMD│  │           │
│  │  └────┘ └────┘ └────┘  │           │
│  └──────────┬───────────────┘           │
│             │                            │
│  ┌──────────▼───────────────┐           │
│  │     Memory Access        │           │
│  └──────────────────────────┘           │
└──────────────────────────────────────────┘
```

## Key Metrics

| Feature | Description | Typical Values |
|---------|-------------|----------------|
| Clock Speed | Cycles per second | 2-5 GHz |
| Cores | Independent processing units | 4-16 (consumer), 64-128 (server) |
| Threads | Logical processors | 2× cores (with Hyper-Threading) |
| IPC | Instructions per cycle | 1-4 |
| TDP | Thermal Design Power | 15-280W |

## Instruction Pipelining

```
Without Pipeline (4 cycles per instruction):
I1: Fetch│Decode│Execute│Write
I2:       │Fetch │Decode │Execute│Write
Total: 8 cycles for 2 instructions

With Pipeline (1 instruction per cycle after fill):
Cycle 1: I1 Fetch
Cycle 2: I1 Decode | I2 Fetch
Cycle 3: I1 Execute | I2 Decode | I3 Fetch
Cycle 4: I1 Write | I2 Execute | I3 Decode | I4 Fetch
```

**Pipeline Hazards:**
- **Data hazard:** Instruction depends on previous result
- **Control hazard:** Branch instruction (may mispredict)
- **Structural hazard:** Two instructions need same resource

### Branch Prediction

Modern CPUs predict whether branches will be taken:
- **Static prediction:** Always/never taken (simple)
- **Dynamic prediction:** Based on history (95%+ accuracy in modern CPUs)
- **Misprediction penalty:** 10-20 cycles of wasted work

> 💡 **Pro Tip:** For performance-critical paths, hint the compiler about branch likelihood using `[[likely]]` / `[[unlikely]]` (C++20) or `__builtin_expect` (GCC/Clang). This helps the CPU's branch predictor make better decisions.

## SIMD (Single Instruction, Multiple Data)

Process multiple data points with one instruction:

```
Without SIMD:   ADD R1, R2 | ADD R3, R4 | ADD R5, R6
With AVX-512:   VADD ZMM1, ZMM2 (16 pairs at once)
```

SIMD use cases: image processing, audio, scientific computing, ML.

> ✅ **Best Practice:** When writing ML inference code, prefer libraries (NumPy, PyTorch, TensorFlow) that use SIMD internally. Manual SIMD intrinsics are error-prone — let the library handle vectorization for you.

## Common Mistakes

1. **Assuming all cores equal:** Heterogenous architectures (ARM big.LITTLE).
2. **Cache-unfriendly algorithms:** Random access patterns destroy performance.
3. **Ignoring branch prediction:** Unpredictable branches in hot paths.

## Interview Questions

1. **Q:** Process vs thread? How does CPU handle them?  
   **A:** Process has own memory space; threads share memory. CPU switches via context switching.

2. **Q:** What is cache coherence in multi-core?  
   **A:** Ensures all cores see the same data. Protocols like MESI maintain coherence.

3. **Q:** Types of cache misses?  
   **A:** Compulsory (first access), Capacity (cache too small), Conflict (mapping limitation), Coherence (invalidation).

## Revision Notes

- CPU: Cores, threads, clock speed, IPC
- Pipelining: Fill pipeline → 1 instruction/cycle
- Cache: L1 (~32KB), L2 (~256KB), L3 (~8-32MB)
- SIMD: Same operation on multiple data
- Branch prediction minimizes pipeline stalls

---

# Chapter 5: Operating Systems

## Introduction

The OS manages hardware and provides services for applications. This chapter covers process management, memory management, and file systems.

## Why It Matters

- Every program runs on top of an OS
- Debugging requires understanding OS behavior
- Containers (Docker) leverage OS virtualization
- Cloud computing is built on OS virtualization

## Real World Analogy

The OS is like a building manager:
- **Kernel = Building manager** (controls everything)
- **CPU Scheduling = Meeting room booking** (who gets time)
- **Memory Management = Apartment allocation** (who gets space)
- **File System = Storage units** (organized storage)

## OS Architecture

```
┌─────────────────────────────────────────┐
│         User Applications              │
│  (Browser, IDE, Games, etc.)          │
├─────────────────────────────────────────┤
│        System Libraries               │
│  (glibc, Win32 API, POSIX)            │
├─────────────────────────────────────────┤
│        System Call Interface           │
├─────────────────────────────────────────┤
│                                         │
│              KERNEL                    │
│  ┌──────┬──────┬──────┬──────┐        │
│  │Process│Memory│File  │Device│        │
│  │Manager│Manager│System│Driver│        │
│  └──────┴──────┴──────┴──────┘        │
│  ┌──────────────────────────────┐      │
│  │        Network Stack         │      │
│  └──────────────────────────────┘      │
├─────────────────────────────────────────┤
│              Hardware                  │
│   (CPU, RAM, Disk, Network, etc.)     │
└─────────────────────────────────────────┘
```

## Process Management

### Process States

```
   NEW → READY → RUNNING → TERMINATED
              ↑       ↓
              ← BLOCKED (I/O)
```

### Context Switching

When CPU switches processes:
1. Save current state (registers, PC)
2. Load saved state of next process
3. Update memory management structures
4. Resume execution

Overhead: 1-10 microseconds (thousands of instructions)

> ⚠️ **Warning:** Creating too many threads causes performance to degrade due to context switching overhead. For CPU-bound tasks, use at most `Runtime.getRuntime().availableProcessors()` threads. For I/O-bound tasks, consider async/event-driven models.

## Memory Management

### Virtual Memory

Each process gets its own virtual address space:

```
Process A:       Process B:
0xFFFFFFFF      0xFFFFFFFF
┌─────────┐     ┌─────────┐
│  Stack  │     │  Stack  │
│    ↓    │     │    ↓    │
│  (gap)  │     │  (gap)  │
│    ↑    │     │    ↑    │
│  Heap   │     │  Heap   │
│  Data   │     │  Data   │
│  Text   │     │  Text   │
└─────────┘     └─────────┘
0x00000000      0x00000000

         │
         ▼
┌──────────────────────┐
│   Physical Memory    │
│  ┌────┬────┬────┐   │
│  │ A  │ B  │ .. │   │
│  └────┴────┴────┘   │
│    Swap (disk)      │
└──────────────────────┘
```

### Paging

Memory divided into fixed-size pages (typically 4KB):

- Virtual address = Page Number + Offset
- Page table maps virtual pages to physical frames
- Page fault = access page not in RAM → load from disk

> ❓ **Interview Question:** What is thrashing? — When the OS spends more time swapping pages between RAM and disk than executing processes. This happens when there's insufficient RAM for the working set. Solution: add more RAM or reduce the number of running processes.

## File Systems

### VFS (Virtual File System)

```
Application (open, read)
    → VFS (abstract interface)
    → Specific FS (ext4, NTFS, APFS)
    → Block Device Layer
    → Storage (SSD/HDD)
```

### Common File Systems

| FS | OS | Features |
|----|-----|----------|
| ext4 | Linux | Journaling, backward compatible |
| NTFS | Windows | Journaling, permissions, encryption |
| APFS | macOS | Snapshots, encryption, space sharing |
| XFS | Linux | High performance, large files |

## Common Mistakes

1. **Not handling OOM conditions:** Apps must handle OutOfMemoryError.
2. **Creating too many threads:** OS has limits on threads per process.
3. **Not understanding buffers/caches:** `free` shows cached memory misleadingly.

## Interview Questions

1. **Q:** Process vs thread?  
   **A:** Process = independent program with own memory. Thread = lightweight unit within process, shared memory.

2. **Q:** Explain virtual memory benefits.  
   **A:** Each process gets own address space, isolation, programs larger than physical RAM, simplified memory management.

3. **Q:** What is a system call? Examples?  
   **A:** How program requests service from kernel. Examples: `open()`, `fork()`, `socket()`, `brk()`.

## Revision Notes

- OS: Kernel + System Libraries + Applications
- Virtual memory: Each process gets 4GB (32-bit) address space
- Paging: 4KB pages, page tables map virtual → physical
- Context switching: 1-10μs overhead
- Process states: New → Ready → Running → Blocked → Terminated

---

# Chapter 6: Linux Fundamentals

## Introduction

Linux powers 96% of web servers, all Android devices, most supercomputers, and the entire cloud. This chapter covers essential Linux knowledge.

## Why It Matters

- Servers: 96% of top million web servers run Linux
- Cloud: AWS, GCP, Azure all run Linux
- DevOps: Docker, Kubernetes, Terraform target Linux
- AI/ML: PyTorch, TensorFlow, CUDA run on Linux

## Linux Philosophy

1. **Everything is a file:** Devices, sockets, pipes accessed as files
2. **Small, focused tools:** Each tool does one thing well
3. **Composability:** Tools chained together (pipes)
4. **Text is universal interface:** Config, logs, data exchange
5. **Configuration as code:** `/etc` contains system config

## File System Hierarchy

```
/              Root
├── /bin       Essential commands
├── /boot      Boot files, kernel
├── /dev       Device files
├── /etc       Configuration
├── /home      User home directories
├── /proc      Process information (virtual)
├── /tmp       Temporary files
├── /usr       User system resources
├── /var       Variable data (logs, databases)
└── /opt       Optional software
```

## File Permissions

```
String:  rwx rwx rwx
          │   │   └── Others
          │   └────── Group
          └────────── Owner

Numeric: r=4, w=2, x=1
  rwx = 7, rw- = 6, r-x = 5, r-- = 4

Example: chmod 755 script.sh
  Owner: rwx (7)
  Group: r-x (5)
  Others: r-x (5)
```

> ⚠️ **Warning:** Never use `chmod 777` — it makes files world-writable and is a security risk. Use `755` for directories and executables, `644` for regular files. On production servers, use more restrictive permissions like `750` or `700`.

## Essential Commands

### File Operations

```bash
ls -la                    # List files with details
cd /path                  # Change directory
pwd                       # Print working directory
cp source dest            # Copy
mv source dest            # Move/rename
rm -rf dir                # Remove recursively
mkdir -p a/b/c            # Create nested directories
cat file.txt              # Display content
less file.txt             # View with scroll
head -n 20 file.txt       # First 20 lines
tail -f file.txt          # Follow (live log)
```

### Process Management

```bash
ps aux                    # List all processes
top / htop                # Interactive process viewer
kill -9 PID               # Force kill
nohup command &           # Run in background, survive logout
```

### Search and Text Processing

```bash
grep -r pattern /dir      # Search recursively
find /dir -name "*.java"  # Find files by name
sed 's/old/new/g' file    # Replace text
awk '{print $1}' file     # Process columns
sort file | uniq -c       # Count unique lines
```

### Network Commands

```bash
ping google.com           # Test connectivity
curl http://example.com   # Transfer data
ssh user@host             # Secure shell
scp file user@host:/path  # Secure copy
dig example.com           # DNS lookup
ss -tulpn                 # Listening ports
```

### Package Management (Ubuntu/Debian)

```bash
sudo apt update           # Update package list
sudo apt install pkg      # Install package
sudo apt remove pkg       # Remove package
dpkg -l                   # List installed packages
```

## Shell Scripting

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backup/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

for dir in /home/user/docs /etc/nginx; do
    if [ -d "$dir" ]; then
        tar -czf "$BACKUP_DIR/$(basename $dir).tar.gz" "$dir"
    fi
done

echo "Backup size: $(du -sh $BACKUP_DIR | cut -f1)"
```

> 💡 **Pro Tip:** Always start your shell scripts with `set -euo pipefail`. This makes them safer: `-e` exits on error, `-u` treats unset variables as errors, and `-o pipefail` catches failures in piped commands. It's saved countless debugging hours.

## Common Mistakes

1. **Running as root:** Use `sudo` for specific commands.
2. **Chmod 777:** Never set world-writable permissions.
3. **`rm -rf /`:** Never run. Always double-check paths.
4. **Not escaping variables:** Use `"$VAR"` not `$VAR` in scripts.

## Interview Questions

1. **Q:** Explain the Linux boot process.  
   **A:** BIOS/UEFI → GRUB → Kernel loads → Init (PID 1) → Systemd → Services → Login.

2. **Q:** Soft link vs hard link?  
   **A:** Hard link = additional directory entry to same inode. Soft link = special file pointing to path. Hard links can't cross filesystems.

## Mini Project: System Monitor

Create a bash script that: CPU usage (top 5), memory usage, disk usage, network activity, logs to file with timestamps, alerts if disk > 80%.

---

# Chapter 7: Networking Basics

## Introduction

Computer networking enables communication between devices. The internet is a global network of networks.

## Why It Matters

- Every application communicates over a network
- Web dev requires understanding HTTP/HTTPS
- Microservices communicate via network protocols
- Cloud computing is network-based computing

## Real World Analogy

The internet is like a postal system:
- **IP Address = Street address** (where to deliver)
- **Port = Apartment number** (which application)
- **DNS = Phone book** (name → address lookup)
- **Packets = Letters** (units of data)
- **TCP = Certified mail** (guaranteed delivery)
- **UDP = Postcard** (fast, no guarantee)

## The OSI Model (7 Layers)

| Layer | Function | Example Protocols |
|-------|----------|------------------|
| 7. Application | User-facing apps | HTTP, FTP, SMTP |
| 6. Presentation | Data format, encryption | SSL/TLS, JPEG |
| 5. Session | Session management | NetBIOS, RPC |
| 4. Transport | End-to-end delivery | TCP, UDP |
| 3. Network | Routing, addressing | IP, ICMP |
| 2. Data Link | Frame transmission | Ethernet, Wi-Fi |
| 1. Physical | Raw bits | Cables, radio |

## TCP/IP Model

```
┌──────────────────────┐
│    Application       │ ← HTTP, HTTPS, DNS, SSH
├──────────────────────┤
│    Transport         │ ← TCP, UDP
├──────────────────────┤
│    Internet          │ ← IP, ICMP
├──────────────────────┤
│    Network Access    │ ← Ethernet, Wi-Fi
└──────────────────────┘
```

## TCP (Transmission Control Protocol)

Connection-oriented, reliable delivery:

**Three-Way Handshake:**
```
Client                     Server
  ├──SYN──────────────►│
  │◄──SYN-ACK──────────┤
  ├──ACK──────────────►│
  └──Data Transfer─────┘
```

**TCP Features:**
- **Reliability:** Retransmits lost packets
- **Ordering:** Reorders out-of-order packets
- **Flow control:** Prevents overwhelming receiver
- **Congestion control:** Adjusts to network conditions

> ❓ **Interview Question:** Why is TCP considered "reliable" when packets can still be lost? — TCP guarantees delivery through acknowledgments and retransmission, detects corruption via checksums, maintains ordering, and prevents congestion collapse. The application sees a reliable byte stream even though the underlying network isn't reliable.

## UDP (User Datagram Protocol)

Connectionless, no guarantees. Use cases:
- DNS queries
- Video streaming (some loss acceptable)
- VoIP (real-time)
- Online gaming

## IP Addressing

```
IPv4: 192.168.1.1 (32-bit)

Private IP Ranges:
  10.0.0.0/8         (10.x.x.x)
  172.16.0.0/12      (172.16.x.x - 172.31.x.x)
  192.168.0.0/16     (192.168.x.x)

CIDR: 192.168.1.0/24 = 256 addresses
```

## DNS (Domain Name System)

```
Browser → Local DNS Resolver → Root Server → .com Server → Authoritative
```

**DNS Record Types:**
- `A` → IPv4 address
- `AAAA` → IPv6 address
- `CNAME` → Canonical name (alias)
- `MX` → Mail exchange
- `TXT` → Text data (SPF, DKIM)

> 💡 **Pro Tip:** DNS resolution adds 10-100ms to request latency. Use DNS caching (e.g., `dnsmasq` locally or a caching resolver) and set appropriate TTL values on your DNS records. For microservices, consider service discovery tools like Consul or Kubernetes DNS.

## HTTP/HTTPS

```http
GET /api/users HTTP/1.1
Host: example.com
Authorization: Bearer token123
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123
{"users": [{"id": 1, "name": "Alice"}]}
```

HTTPS = HTTP over TLS (encrypted).

## Common Mistakes

1. **Not handling network failures:** Networks are unreliable.
2. **Hardcoding IP addresses:** Use DNS names.
3. **Not understanding NAT:** Private IPs unreachable from internet.
4. **Ignoring latency:** Each network hop adds delay.

## Best Practices

- Always use HTTPS in production
- Implement retry logic with exponential backoff
- Use connection pooling for databases
- Monitor network latency and packet loss

## Interview Questions

1. **Q:** What happens when you type a URL in a browser?  
   **A:** Cache check → DNS lookup → TCP (SYN/SYN-ACK/ACK) → TLS handshake (HTTPS) → HTTP request → Server processing → HTTP response → Render.

2. **Q:** TCP vs UDP?  
   **A:** TCP: connection-oriented, reliable, ordered, flow/congestion control. UDP: connectionless, unreliable, unordered, less overhead.

---

# Chapter 8: REST, GraphQL, and WebSockets

## Introduction

Modern web applications communicate via APIs. REST dominates, GraphQL offers flexibility, WebSockets enable real-time communication.

## Real World Analogy

- **REST = Restaurant menu** (fixed dishes, each URL is a menu item)
- **GraphQL = Buffet** (you pick exactly what you want)
- **WebSocket = Phone call** (persistent two-way conversation)

## REST

### REST Constraints

1. **Stateless:** Each request contains all needed info
2. **Client-Server:** Separation of concerns
3. **Cacheable:** Responses can be cached
4. **Uniform Interface:** Standard resource identification
5. **Layered System:** Proxies, gateways allowed
6. **Code on Demand (optional):** Executable code from server

### Resource Naming

```
GET    /users           → List users
POST   /users           → Create user
GET    /users/{id}      → Get user
PUT    /users/{id}      → Update (full)
PATCH  /users/{id}      → Update (partial)
DELETE /users/{id}      → Delete
```

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Server Error |

> 💡 **Pro Tip:** Memorize these status code categories: 2xx (success), 3xx (redirection), 4xx (client error), 5xx (server error). In production, ensure your API never leaks 500 error details to clients — log them server-side and return a generic message.

## GraphQL

Query language for APIs — clients specify exactly what data they need:

```graphql
query {
  user(id: "1") {
    name
    email
    posts { title }
  }
}
```

> ✅ **Best Practice:** Use GraphQL when your clients need flexible data fetching and you have complex, nested data relationships. Use REST when your API is simpler, you want maximum cacheability, or you need broad tooling support. Many teams use both: REST for simple CRUD, GraphQL for complex queries.

## WebSockets

Persistent, full-duplex connection:

```
Client → HTTP Upgrade Request → Server (101 Switching Protocols)
        ← Bidirectional messages →
        ← Close Frame →
```

> ❓ **Interview Question:** When would you use WebSockets instead of HTTP polling? — WebSockets are ideal for real-time applications (chat, live notifications, collaborative editing, live dashboards, online gaming) where low latency and server-initiated messages are needed. HTTP polling wastes bandwidth with repeated request/response cycles.

## Comparison

| Feature | REST | GraphQL | WebSocket |
|---------|------|---------|-----------|
| Transport | HTTP | HTTP | WS/WSS |
| Over-fetching | Common | Avoided | N/A |
| Under-fetching | Common | Avoided | N/A |
| Real-time | Polling | Subscriptions | Native |
| Best for | CRUD APIs | Complex data | Live updates |

## Best Practices

- Use OpenAPI/Swagger for REST docs
- Implement rate limiting
- Use pagination for lists
- Version your APIs
- Use HTTPS everywhere

---

# Part 1 Summary

## Key Concepts Mastered

- Von Neumann architecture: CPU + Memory + Storage
- Number systems: Binary, Decimal, Hex, conversions
- Data representation: Text (ASCII, UTF-8), Images, Audio
- CPU: Pipelining, Cache, SIMD, Branch Prediction
- OS: Processes, Virtual Memory, File Systems
- Linux: File hierarchy, commands, permissions, scripting
- Networking: OSI, TCP/IP, DNS, HTTP/HTTPS
- APIs: REST, GraphQL, WebSockets

## Revision Questions

1. Explain the Fetch-Decode-Execute cycle.
2. Convert 2026 decimal to binary and hex.
3. What is the difference between TCP and UDP?
4. Explain virtual memory and paging.
5. What are the REST constraints?
6. How does DNS resolution work?
7. What is the Linux file permission format?
8. Explain CPU cache levels.

## Projects to Reinforce

1. Build a number system converter CLI
2. Write a system monitoring bash script
3. Set up a web server with Nginx
4. Create a REST API with Express/Spring Boot
5. Implement a simple TCP chat server

## References

- [Von Neumann Architecture](https://en.wikipedia.org/wiki/Von_Neumann_architecture)
- [IEEE 754 Standard](https://ieeexplore.ieee.org/document/8766229)
- [Linux Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs-3.0.pdf)
- [IBM TCP/IP Tutorial](https://www.ibm.com/docs/en/zos-basic-skills?topic=concepts-tcpip)
- [REST Architectural Style](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
