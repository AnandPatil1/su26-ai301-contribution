# Contribution 1: Implement missing POOL_1D op for Vulkan

**Contribution Number:** 1  
**Student:** Anand Patil  
**Issue:** [Feature Request: Implement missing ops from backends
 #14909](https://github.com/ggml-org/llama.cpp/issues/14909)  
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because my goal is to understand large language model inferences, and this issue dives into that along with tying into hardware backends. I'm also interested in the llama.cpp codebase since it is a high-performance environment, and this opportunity to implement a specific operation for the Vulkan backend lets me become an engine contributor.

I specifically selected the POOL_1D operation because it is a clearly defined, bounded operation, and if I analyze the existing POOL_2D implementation in the Vulkan backend, I have a strong reference pattern to follow. From this issue, I hope to learn a bit about GPU programming and gain skills with Vulkan.

---

## Understanding the Issue

### Problem Description

The POOL_1D (1D Pooling) operation is currently marked as unsupported for the Vulkan backend in the ops.md documentation. This means that any models requiring 1D pooling operations will fail to run when using the Vulkan backend.

### Expected Behavior

When test-backend-ops -o POOL_1D --backend vulkan is run, the operation should report "Passed" or "OK," indicating that the Vulkan kernel for POOL_1D is correctly implemented and verified against the CPU reference implementation.

### Current Behavior

Running the test tool reports "Not supported," meaning that the implementation is missing from the Vulkan backend.

### Affected Components

src/ggml-vulkan.cpp: The primary file handling Vulkan backend dispatch and kernel logic.

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
