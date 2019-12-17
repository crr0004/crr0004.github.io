title:
date:
tags:
---
# Introduction

This is in continuation from {% post https://dev.to/crr0004/opencl-the-wrong-way-part-2-1-2m0c-temp-slug-3996427 %} This is a working version of summing numbers, that is another sub-problem of what we were doing before.

# Problem Statement

As always, a problem statement is Step 1. Given a list of numbers, sum all numbers in the list.

For this next post we are going to focus on turning the sub problem of adding a list of numbers together in a parallel way. It is much like the previous problem, however we only care about the final sum, not the intermediate sums. You may be asking why I am explaining such a simple task, however as it turns out it is quite complex to do and we can learn a lot. When we all first started programming, we dealt with simple problems and learnt much from them, we are doing the same for parallel computing.


# Solution

If we take the property that a sum can be computed in any order, that is `a + b = b + a`, we can reason that it doesn't matter about what order the OpenCL kernel execute in, which is one part that blocked us before. We do still have a data dependency issue of to compute the final sum, we must compute the smaller parts of the sum. This is often the case, we need everything to complete in one step before we can move onto the next.

So we could compute each separate part of the sum, and then at some point do the last part of the sum. If we have the numbers `1, 2, 3, 4, 5, 6`, we could compute `1 + 2 + 3 + 4` in one part, and `5+6` in another, then we are left with `7+11`. Some of you may recognise this as a form of map and reduce functions. The issue is how much do we split our data, and when do we do the reduce part. This is ultimately an optimisation problem and if we parameterise our code correctly, we can do profiling and build a correlation to determine what is going on. The most optimal solution will actually change depending on the hardware, especially with ensuring we our compute bound, not memory bound. 

GPUs have different memory models to CPUs and perform differently under different circumstances, we need to keep this in mind when determining how to split up our data to be computed on. While this is an optimisation, and I don't encourage you to try to find the right combination at the beginning, just keep in mind that different compute combinations yield different results. Try to keep things simple when first designing solutions. The following diagram is visual representation of one solution.

![solution diagram](https://i.imgur.com/6Ee3L6a.png)

# Kernel Breakdown

In this section I am going to break down the [source code from one the kernels](https://raw.githubusercontent.com/crr0004/adventures-in-opencl/master/summing_numbers/kernel.cl) I have written. Most of the code that is used to set this up and run is the same for different kernels.

```
__kernel void add(__global int *numbers, __global int *numOut, __local int *numReduce) {

	// Get the index of the current element to be processed
	int i = get_global_id(0)*2;
	int locali = get_local_id(0);
	int2 va = vload2(i, numbers);
	int2 vb = vload2(i+1, numbers);
	int2 vc = va + vb;
	numReduce[locali] = vc[0] + vc[1];

	printf(
			"%d\t%d: %d + %d + %d + %d = %d\n", 
			i, 
			locali,
			va[0], va[1], vb[0], vb[1], 
			numReduce[locali]
	      );

}
```

## Header Signature
`__kernel void add(__global int *numbers, __global int *numOut, __local int *numReduce)`.
- `__kernel` marks the function that it can be invoked by the device
- `__global` marks the pointer that is part of the global memory
- `__local` marks the pointer that it is part of local memory

## Memory Types
### Global
A type of memory that all the kernels have read and write access to, as well as the host. The host allocates it. We use to read and write the integers into the kernels.

### Local
A type of memory that only kernels within a work group have read and write access to, the host only as allocates it. Kernels can static allocate it though. We use it to hold the reduced numbers into, so we can due further computation on it.

## Function Body
In respect to lines:
1. `int i` - We grab the index of the array we are operating on, the `*2` is because we are expecting each kernel to operate on two pieces, or 4 numbers.
2. `int locali` - We grab the local index of the workgroup so the work item can store its results.
3. `int2 va` - We grab two numbers from the array and store them in a fixed size array of 2
4. `int2 vb` - Same for `int2 va` but we want the two numbers after `va`
5. `int2 vc` - Add the elements of each array. This is NOT adding each number sequentially, but operates similar to SIMD. The actual vendor implementation determines if SIMD will actually be used. 
6. `numReduce[locali]` - Add the result of the previous addition and then store that result in the memory allocated for this work item
7. `printf` - Print out the results we just computed

## Making this useful
For the astute reader will notice that this isn't particularly useful and we would need to get the result of all the numbers added together. So the next step would either to be keep going in this adding process, or to output the immediate steps and then compute the sum somewhere else. As we're outputting the immediate results to local memory, you want to have each kernel wait until the other kernels are done, and then continue to add the numbers together.

# SIMD Aside

I want to quickly introduce the idea of Single Instruction Multiple Data (SIMD) here as it plays an important part in creating a mental model of how data is mapped into memory and then operated on. SIMD is a CPU (GPUs have it as well) instruction set which allows you to apply the same operation on multiple pieces of data. It does this by putting a chunk of data into a large register (128b, 256b, 512b, etc) and then operating on it. In source code this often looks like applying operations (add, multiple, swap, shift, etc) on fixed sized arrays. The following code is a simplified version of adding two sets of numbers together.

```
uint4 a = {1, 2, 3, 0}
uint4 b = {9, 5, 2, 7}

uint4 c = a + b
# c is now {10, 7, 5, 7}

```

The import part to realise here is that the adding operation will happen on all pairs of data at once, and won't happen separately. This quick aside is meant to quickly demonstrate that the time complexity of programs is often very to the algorithmic complexity and we need to pay special attention to how we do this in OpenCL as things are not as they seem.

# Conclusion

When looking at trying to create parallel algorithms it's important to pay attention to what hardware you're working on, and how your memory is going to be layed out. While cache optimisation is important, I purposely haven't touched on it in this post, it's more important to understand how you're going to map your memory to your compute resources.

# Next Time

In the next post I will be going how to solve the painters partition problem, with special attention to transforming a recursive solution into a parallel one. 
