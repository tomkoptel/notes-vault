#interview
# General Advice
1. Solve the tasks on your own and consider runtime and space complexity. Don't tunel yourself in the known subset of runtimes. Derive the values for runtime and space complexity, don't guess those.
2. Write the code on the paper. Should help to get used to the process and how slow it is.
3. Test your coe on the paper with the general cases, base cases, error cases.
4. Try to do as many mock interviews as possible.
# Must have knowledge
Data Structures
* Linked List
* Trees, Tries & Graphs
* Stacks & Queues
* Heaps
* Vectors/ArrayLists
* Hash Tables
Algorithms
* Breadth-First Search
* Depth-First Search
* Binary Search
* Quick Sort
Concepts
* Bit Manipulation
* Memory (Stack vs. Heap)
* Recursion
* Dynamic Programming
* Big O Time & Space
# Walk Throug a Problem
1. Listen. All info might be necessary to come up with the optimal solution.
2. Example. Think about special case examples. Question the size of example? Maybe it is too small and thus hard to see the bigger picture (e.g. drawing 2 level balanced tree).
3. Brute Force. Should be the first naive solution. It helps to warm up.
4. Optimize.
	1. Walk through with BUD and try to optimize or;
	2. Look for some unused info, it can hint on the solution or;
	3. Solve it manually and try reverse engineer. How did you come up with an approach? or;
	4. Solve it "incorrectly". Why did it fail? What can be done to fix it? or;
	5. Think about time vs space tradeoffs. Hash table can be a good pick.
5. Implement. As you implement modularise code and avoid dirty statements. Use comment for weird conditions.
6. Test.
	1. Conceptual test. Work through your code.
	2. Pay attention to unusual or non-standard code. Is it a bug? Is it expected?
	3. Check for common hot spots (NPE, zero divisions, index out of bound).
	4. Special cases
# Optimize and Solve #1: BUD
* Bottlenecks
	* Look for the steps in algorithm. Look for steps that take most of the time. Optimize those before moving to another step of algorithm.
* Unnecessary work
	* Question the code. Do we need to repeat this action this many times? Think about the goal to be achieved.
* Duplicated work
Those are common things to look in algorithms to avoid the wasted work.
# Optimize and Solve #2: DIY
Try to imagine approaching the problem manually with the hands. Think about redundant work you would try to avoid by simply being "lazy".
# Optimize and Solve #3: Simplify and Generalize
Tweak the original data structure to come up with the algorithm. With algorith in hands generalize (e.g. instead of working with words, first work with characters).
# Optimize and Solve #4: Base Case and Build
Solve a problem for a base case (e.g. n = 1) then and then try to build up from then. For example, compute all permutations of "abc".
* n = 1. "a"
* n = 2. {"ab"}, {"ba"}
* n = 3. {"cab", "acb", "abc"}, {"cba", "bca", "bac"}
This approach naturally leads us to the recursive solution.