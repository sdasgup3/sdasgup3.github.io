---
layout: archive
title: "Projects"
permalink: /projects/
author_profile: true
redirect_from: 
  - /projects
---

Clang based AST Instrumentation [Github](https://github.com/sdasgup3/profiler-using-clang-based-ast-instrumentation)
-------------------------------
**Abstract:** Clang based AST reformatting tool used for injecting
instrumentation code in C/C++ programs. The goal is collecting profiles
(run-times and execution frequencies) on C/C++ programs.  

LLVM based Dwarf Type Reader [Github](https://github.com/sdasgup3/dwarf-type-reader)
----------------------------
**Abstract:** To read type information from debug info section of executable
using LLVM based APIs.

Pointer Analysis Debugger [Github](https://github.com/sdasgup3/symbolic-analysis) [PPT](../files/pa_debugger.pdf)
------------
**Abstract:** Finding bugs in LLVM's pointer analysis using a mix of static
analysis and symbolic execution (using KLEE).

Partial Redundancy Elimination(PRE) [Github](https://github.com/sdasgup3/PartialRedundancyElimination) [Report](../files/report_cs526.pdf)
------------
**Abstract**: PRE is a compiler optimization that eliminates expressions that
are redundant on some but not necessarily all paths through a program. In this
project, we implemented a PRE optimization pass in LLVM and measured results on
a variety of applications. 

Implementing "Non-Separable" Data-Flow-Analyzer [Github](https://github.com/sdasgup3/NonSeparableGlobalDataFlowFramework) [Report](../files/report_gdfa.pdf)
------------------
**Abstract**: To Extend the Generic Data Flow Analyzer GDFA (of gcc) to the
data flow frameworks where data flow information can be represented using bit
vectors but the frameworks are not bit vector frameworks because they are
non-separable e.g., faint variable analysis, possible undefined variable
analysis, strongly live variable analysis.	

Designing Interpreter for a dynamic language for Graph Algorithms [Github](https://github.com/sdasgup3/gri) [Report](../files/report_cs598dhp.pdf)
------------
**Abstract**: Designed a dynamically typed language and an interpreter for it
and achieved a slowdown of 2X w.r.t the execution time of statically compiled C
language. This is obtained by providing built-in compiled functions for simple
graph computation which in turn help to build complex ones.

Mitigating Impact of Heterogeneity Across Power-constrained Nodes on Parallel Applications through Load Balancing [Github](https://github.com/sdasgup3/HeterogeneityAwareLoadBalancing) [Report](../files/report_hetero.pdf)
----------------
**Abstract**: Different processors across the nodes have different execution
times for the same work-loads. This performance imbalance is seen only when the
CPU power is capped to low values. This performance imbalance causes increased
execution times of the parallel applications. We did a detailed study and
proposed a power aware load balancer (using Charm++ ) which minimized the
performance imbalance at the lower power caps by tackling this heterogeneity.  

Designing Superscalar Processor [Github](https://github.com/sdasgup3/Parallel-Processor-Design)
------------
**Abstract**: To design a customized processor (using parallel processing
    concepts) for the application of document retrieval system. We developed a
superscalar processor (with  an issue rate of 2) using verilog hdl, and an
assembler for that processor using flex and bison. 

Graph Coloring Using State Space Search [Github](https://github.com/sdasgup3/ParallelSudoku) [Report](../files/CS598_project_proposal.pdf)
------------
**Abstract**: We plan to leverage the state space search model for implementing
graph coloring in parallel in Charm++. Some of the challenges for efficient
exploration of space by chares include intelligent pruning of the state space,
            load balancing, grain-size control and low-overhead communication
            between chares. We evaluated multiple options for each of these,
            and come up with design decisions which would work for a large
            category of real-life graph applications.


## Graduate Courses
 - [Scripting Languages - Design and Implementation](http://polaris.cs.illinois.edu/CS598)
 - [Advanced Compiler Construction](https://cs.illinois.edu/courses/profile/cs526/)
 - [Parallel Computer Architectures](https://courses.engr.illinois.edu/cs533/)
 - [Parallel Programming with Migratable Objects](https://wiki.cites.illinois.edu/wiki/display/cs598lvk/Home)
 - [Introduction to Parallel Programming for Scientists and Engineers](https://cs.illinois.edu/courses/profile/CS420)

## MTech Thesis (Advisor [Dr. Amey Karkare](http://www.cse.iitk.ac.in/users/karkare)) [Report](http://www.cse.iitk.ac.in/users/karkare/MTP/2010-11/sandeep2010precise.pdf)
**Precise Shape Analysis Using Field Sensitivity**: To disambiguate heap
allocated data-structures by estimating the shape (Tree, Dag or Cyclic Graph )
of the data structure accessible from each heap directed pointer. This will
help in automatic parallelization of sequential code having heap intensive data
structures.  The work mainly focuses on devising a novel shape analysis
technique.
