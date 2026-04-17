# Tachyon Pulse (v1.0)

### A Lightweight Silicon Integrity & Throughput Audit

**Tachyon Pulse** is a high-entropy performance benchmark designed to measure the raw sequential processing speed of a CPU while simultaneously auditing memory integrity. By simulating a 4K image buffer (8.2 million pixels) and performing a "Pixel-for-Pixel" (P-for-P) verification, it identifies thermal throttling, core-scheduling shifts, and potential bit-flips in the memory subsystem.

---

## Line-by-Line Logic Breakdown

This is how the "Ghost" logic operates within the script:

#### 1. The Entropy Engine
```cpp
uint32_t xorshift32(uint32_t &state) {
    state ^= state << 13;
    state ^= state >> 17;
    state ^= state << 5;
    return state;
}

```
Instead of using standard random functions which are slow, we use **Xorshift32**. This creates high-entropy data using only bitwise shifts and XOR operations. This ensures the CPU is fed unpredictable data, preventing "branch prediction" shortcuts.
#### 2. The Buffer Allocation
```cpp
const size_t sector_size = 8294400; // 4K Frame
std::vector<uint32_t> pixels(sector_size);

```
We allocate a contiguous block of memory equivalent to a 4K RGBA image (~32MB). This is large enough to exceed the L1 and L2 caches of most mobile CPUs, forcing the system to utilize the **L3 cache and System RAM**.
#### 3. The Search Phase (The Pulse)
```cpp
volatile size_t matches = 0; 
for (size_t i = 0; i < sector_size; ++i) {
    if (pixels[i] == target) { matches++; }
}

```
The volatile keyword is the "Ghost" in the logic—it prevents the compiler from optimizing the loop away. The CPU is forced to physically inspect every single pixel in the 8.2MP buffer.
#### 4. The P-for-P (Pixel-for-Pixel) Audit
```cpp
for (size_t i = 0; i < sector_size; ++i) {
    if (pixels[i] != xorshift32(verify_state)) { mismatch_count++; }
}

```
This is the most critical part of the script. We reset the entropy seed and re-calculate the entire 4K frame. If pixels[i] does not match the re-calculated value, it means a **bit-flip** occurred or the memory was corrupted during the high-speed search.
## How to Compile & Run
To achieve the "Tachyon" speeds seen in the benchmark (e.g., ~4ms per 4K frame), you must compile with aggressive optimization:
```bash

# Compile
g++ -O3 ghost_bench5.cpp -o tachyon_pulse

# Run
./tachyon_pulse

```
## Interpreting Results
 * **Search Time:** Represents the throughput of your memory bus and CPU.
 * **Target Matches:** Usually 0 (given the math), used here to force the CPU to look at every bit.
   
 * **Lost/Erroneous Pixels:** Should always be **0**. Any number higher than 0 indicates hardware instability, overheating, or a failure in the P-for-P integrity check.
 * **Outliers:** If you see a sudden jump in speed (e.g., 8ms down to 4ms), you are witnessing the CPU scheduler moving the task to a high-performance "Big" core.
## Project Status
**Current Version:** 1.0 (Basic Pulse)
**License:** CC BY-NC-SA 4.0 (Attribution-NonCommercial-ShareAlike)

### Release Notes

This project serves as a baseline for anyone building performance sensitive software in environments like Termux or localized AI frameworks. It ensures that before you build complex systems, the foundation (your hardware and memory) is stable and fast.
