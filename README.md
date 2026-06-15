# Contribution 1: Implement missing POOL_1D op for Vulkan

**Contribution Number:** 1  
**Student:** Anand Patil  
**Issue:** [Feature Request: Implement missing ops from backends
 #14909](https://github.com/ggml-org/llama.cpp/issues/14909)  
**Status:** Phase II Complete

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
ggml_vulkan: Found 1 Vulkan devices:
ggml_vulkan: 0 = Intel(R) Graphics (Intel Corporation) | uma: 1 | fp16: 1 | bf16: 0 | warp size: 32

Backend 1/2: Vulkan0
  Device description: Intel(R) Graphics

  POOL_1D(pool_type=avg,type_input=f32,ne_input=[10,3,2,1],k0=1,s0=1,p0=0): not supported [Vulkan0]
  POOL_1D(pool_type=avg,type_input=f32,ne_input=[11,1,3,2],k0=1,s0=1,p0=0): not supported [Vulkan0]
  POOL_1D(pool_type=avg,type_input=f32,ne_input=[128,2,1,3],k0=1,s0=1,p0=0): not supported [Vulkan0]
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

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
