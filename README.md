# Contribution 1: Implement missing POOL_1D op for Vulkan

**Contribution Number:** 1  
**Student:** Anand Patil  
**Issue:** [Feature Request: Implement missing ops from backends
 #14909](https://github.com/ggml-org/llama.cpp/issues/14909)  
**Status:** Phase IV Complete

---

## Why I Chose This Issue

I chose this issue because my goal is to understand large language model inferences, and this issue dives into that along with tying into hardware backends. I'm also interested in the llama.cpp codebase since it is a high-performance environment, and this opportunity to implement a specific operation for the Vulkan backend lets me become an engine contributor.

I selected the POOL_1D operation because it is a clearly defined, bounded operation. If I analyze the existing POOL_2D implementation in the Vulkan backend, I have a strong reference pattern to follow, which makes this task an ideal first contribution. From this issue, I hope to learn a bit about GPU programming and gain skills with Vulkan.

---

## Understanding the Issue

### Problem Description

The POOL_1D (1D Pooling) operation is currently marked as unsupported for the Vulkan backend in the ops.md documentation. This means that any models requiring 1D pooling operations will fail to run when using the Vulkan backend.

### Expected Behavior

When test-backend-ops -o POOL_1D --backend vulkan is run, the operation should report "Passed" or "OK," indicating that the Vulkan kernel for POOL_1D is correctly implemented and verified against the CPU reference implementation.

### Current Behavior

Running the test tool reports "Not supported," meaning that the implementation is missing from the Vulkan backend.

### Affected Components

ggml/src/ggml-vulkan/ggml-vulkan.cpp: The primary file handling Vulkan backend dispatch and kernel logic.

---

## Reproduction Process

### Environment Setup

Setted up on Windows using Git Bash, installing the Vulkan SDK (v1.4.350) and CMake, then built the project by running (in Git Bash):
```
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release
```
The main issue I ran into was a "Could NOT find Vulkan" error on my first cmake run. I fixed it by clearing CMakeCache.txt, restarting terminal, and re-running cmake after to make sure the Vulkan SDK's environment variables were registered.

I confirmed Vulkan was working by running (in Git Bash):
```
build/bin/Release/test-backend-ops -o POOL_1D
```
which showed all POOL_1D cases returning not supported [Vulkan0], confirming the op is unimplemented and ready to work on.

### Steps to Reproduce

1. Clone the repo and navigate to the `llama.cpp` root directory
2. Build with Vulkan enabled:
   - `cmake -B build -DGGML_VULKAN=ON`
   - `cmake --build build --config Release`
3. Run the test for POOL_1D: `build/bin/Release/test-backend-ops -o POOL_1D`
4. **Expected:** All POOL_1D test cases pass on the Vulkan backend
5. **Actual:** All test cases return `not supported [Vulkan0]` — 0/0 tests run, op is unimplemented

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/AnandPatil1/llama.cpp/tree/fix-issue-14909-pool1d-vulkan](https://github.com/AnandPatil1/llama.cpp/tree/fix-issue-14909-pool1d-vulkan)
- **Screenshots/logs:** Log of output showing test cases returning "not supported [Vulkan0]" on my Intel integrated GPU (full log available on request):
```
$ build/bin/Release/test-backend-ops -o POOL_1D

Backend 1/2: Vulkan0
  Device description: Intel(R) Graphics

  POOL_1D(pool_type=avg,type_input=f32,ne_input=[10,3,2,1],k0=1,s0=1,p0=0): not supported [Vulkan0]
  POOL_1D(pool_type=avg,type_input=f32,ne_input=[11,1,3,2],k0=1,s0=1,p0=0): not supported [Vulkan0]
  ...
  POOL_1D(pool_type=max,type_input=f32,ne_input=[128,2,1,3],k0=3,s0=2,p0=1): not supported [Vulkan0]

  0/0 tests passed
  Backend Vulkan0: OK
Backend 2/2: CPU
  Skipping CPU backend
2/2 backends passed
OK
```
- **My findings:** POOL_1D is entirely unimplemented in the Vulkan backend as all 48 
  test cases across both avg and max pool types, with varying kernel sizes, 
  strides, and padding return "not supported". The CPU backend skips 
  since it already has support. This confirms the op needs to be added from scratch.

---

## Solution Approach

### Analysis

POOL_1D is missing from the Vulkan backend entirely. There is no kernel implementation or no case-handling in the dispatch logic in `ggml/src/ggml-vulkan.cpp`. The op exists in the GGML op list and works on CPU, but the Vulkan backend simply never handles it.

### Proposed Solution

I will implement POOL_1D for the Vulkan backend by mirroring the existing POOL_2D structure, reducing it from 2D to 1D logic.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The Vulkan backend has no handler for GGML_OP_POOL_1D, so every call to it returns as not supported. Since the CPU backend already handles it correctly, it just needs a Vulkan path.

**Match:** I will use the existing POOL_2D implementation in `ggml/src/ggml-vulkan/ggml-vulkan.cpp` as the primary reference for the pooling logic (avg/max) and kernel/stride/padding parameters.


**Plan:**
1. Add `GGML_OP_POOL_1D` case to the main dispatch logic in `ggml-vulkan.cpp`
2. Add POOL_1D support to the Vulkan backend by porting pool_2d logic, including registry mapping in `ggml-vulkan.cpp` and shader implementation
3. Verify with `build/bin/Release/test-backend-ops -o POOL_1D`. All test cases should pass

**Implement:** https://github.com/AnandPatil1/llama.cpp/tree/fix-issue-14909-pool1d-vulkan

**Review:** I will self-review against the project's contribution guidelines and ensure naming/code style is consistent before opening a PR.

**Evaluate:** The fix will be verified by running the `test-backend-ops` command again. The "not supported" error should be replaced with "Passed." Will also confirm the 2D implementation is untouched by running `test-backend-ops -o POOL_2D`.

---

## Testing Strategy

### Unit Tests

- Test cases: Three input shapes { 10,3,2,1 }, { 11,1,3,2 }, { 128,2,1,3 } covering different length, channel, and batch combinations, each tested under:
  - avg and max pooling with k0 = {1, 3}
  - strides s0 = {1, 2}
  - padding p0 = {0, 1}

### Integration Tests

- Run test-backend-ops for Pool1D on Vulkan backend

### Manual Testing

Ran `build/bin/Release/test-backend-ops -o POOL_1D` on Vulkan backend (Intel Graphics). Got 48/48 test cases passing. Here is the result log:
```
Testing 2 devices

Backend 1/2: Vulkan0
  Device description: Intel(R) Graphics

  POOL_1D(pool_type=avg,type_input=f32,ne_input=[10,3,2,1],k0=1,s0=1,p0=0): OK
  POOL_1D(pool_type=avg,type_input=f32,ne_input=[11,1,3,2],k0=1,s0=1,p0=0): OK
  ...
  POOL_1D(pool_type=max,type_input=f32,ne_input=[128,2,1,3],k0=3,s0=2,p0=1): OK
  48/48 tests passed
  Backend Vulkan0: OK
Backend 2/2: CPU
  Skipping CPU backend
2/2 backends passed
OK
```

---

## Implementation Notes

### Week 3 Progress (June 15-23)
**What I built:**
- Added `vk_op_pool1d_push_constants` struct in `ggml-vulkan.cpp`
- Added `pipeline_pool1d_f32` field to `vk_device_struct` in `ggml-vulkan.cpp`
- Created `pool1d.comp` shader in `ggml/src/ggml-vulkan/vulkan-shaders/`:
  - Removed height dimension and IH/OH/k1/s1/p1 from push constants
  - Reduced nested height/width loops to single length loop
  - Updated index decomposition from `(nc, cur_oh, cur_ow)` to `(nc, cur_ol)`
  - Changed scale from `k0*k1` to just `k0` for 1D average
- Added pipeline creation, selection, supports_op in `ggml-vulkan.cpp`
- Added `ggml_vk_pool_1d` dispatch function
- Added element count case in workgroup sizing switch
- Added POOL_1D case in compute operation dispatch
- Added capture_node handling for graph validation

**Challenges faced:**
- Initially confused on how to remake equations from 2D to 1D
- Confirmed op_params layout by checking `ggml_pool_1d` definition in `ggml/src/ggml.c` to match parameter order

**Decisions made:**
- Followed pool2d as a direct template throughout to stay consistent with code

### Week 4 Progress (June 24-30)

**What I built:**
- Fixed build error by registering `pool1d_f32` shader in `vulkan-shaders-gen.cpp`
- Fixed wrong tensor dimension indices in element count switch and `ggml_vk_pool_1d`
- Fixed `pool1d.comp` shader:
  - Changed `bl`/`el` from `uint` to `int` to safely handle negative start values
  - Changed avg pool scale to divide by actual valid elements in window instead of full kernel size
- Ran `test-backend-ops -o POOL_1D` and passed 48/48 test cases on Vulkan backend

**Challenges faced:**
- Build failed with `pool1d_f32_len` and `pool1d_f32_data` undeclared. I traced it to missing field in `vulkan-shaders-gen.cpp`
- Tests were failing with large errors and info mismatch. I traced it to typo in code and incorrect calculations

**Decisions made:**
- Cross-referenced outer batch layout requirements and CPU reference behavior to fix code issues

### Code Changes

- **Files modified:** ggml/src/ggml-vulkan/ggml-vulkan.cpp, ggml/src/ggml-vulkan/vulkan-shaders/vulkan-shaders-gen.cpp
- **Files added:** ggml/src/ggml-vulkan/vulkan-shaders/pool1d.comp
- **Key commits:**
  - 75a600d: vulkan : add pool1d push constants and pipeline field
  - e2fe6a0: vulkan : add pool1d compute shader
  - 9afd4e0: vulkan : add full GGML_OP_POOL_1D support
  - 1e056c5: vulkan : fix pool1d shader logic
- **Approach decisions:** I chose to follow the existing pool2d implementation as a direct template throughout to stay consistent with existing backend patterns and minimize risk of introducing new behavior.

---

## Pull Request

**PR Link:** [vulkan: add POOL_1D op #25431](https://github.com/ggml-org/llama.cpp/pull/25431)

**PR Description:** 

> ## Overview
> 
> Add the POOL_1D operation for the Vulkan backend.
> 
> ## Additional information
> 
> Part of #14909
> 
> Tested on Intel(R) Graphics:
> 
> ```
>   POOL_1D(pool_type=avg,type_input=f32,ne_input=[10,3,2,1],k0=1,s0=1,p0=0): OK
>   POOL_1D(pool_type=avg,type_input=f32,ne_input=[11,1,3,2],k0=1,s0=1,p0=0): OK
>   ...  
>   POOL_1D(pool_type=max,type_input=f32,ne_input=[128,2,1,3],k0=3,s0=2,p0=1): OK
>   48/48 tests passed
>   Backend Vulkan0: OK
> Backend 2/2: CPU
>   Skipping CPU backend
> 2/2 backends passed
> OK
> ```
> 
> ## Requirements
> 
> - [x] I have read and agree with the [contributing guidelines](https://github.com/ggml-org/llama.cpp/blob/master/CONTRIBUTING.md)
> - AI usage disclosure: YES, I used AI to understand the codebase and to help debug issues I ran into (i.e. missing tensor dimension indices and an error in avg pool scale calculation). I reviewed and tested every change.

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** Awaiting review <!-- / Iterating / Approved / Merged -->

---

## Learnings & Reflections

### Technical Skills Gained

- Deeper understanding of the Vulkan compute backend.
- Learned how to utilize internal verification test suites.

### Challenges Overcome

- Understanding the codebase was difficult, ao I utilized AI to help explain how existing implementations worked.
- Reading through long compiler logs every time a build failed was time consuming, so I utilized AI to help quickly parse errors and identify the root cause.

### What I'd Do Differently Next Time

Next time I would map out all the files I need to work with before starting. One of my major roadblocks was missing an error that turned out to be in a separate file that I didn't realize was needed until the build failed. If I had scoped out the full set of files involved upfront, I would have spent less time navigating and debugging and could have worked through the implementation more efficiently.

---

## Resources Used

- [llama.cpp Contributing Guidelines](https://github.com/ggml-org/llama.cpp/blob/master/CONTRIBUTING.md)
- [llama.cpp Build Documentation](https://github.com/ggml-org/llama.cpp/blob/master/docs/build.md)
