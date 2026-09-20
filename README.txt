COMP 313/413 Project 2 Report
Brendan Berkmans

TestList.java and TestIterator.java

	TODO also try with a LinkedList - does it make any difference?

		No behavioral difference. All 12 tests in TestList and all 4 tests in
		TestIterator pass identically whether the field is initialized as
		new ArrayList<Integer>() or new LinkedList<Integer>(). Both classes
		implement the List interface, and the tests only call methods declared
		on that interface, so the two implementations are interchangeable here.
		The difference between them is performance, not behavior, which is what
		TestPerformance measures.

TestList.java

	testRemoveObject()

		list.remove(5); // what does this method do?

			Removes the element at INDEX 5. List declares two overloaded remove
			methods, remove(int index) and remove(Object o). The literal 5 is an
			int, so Java connected to the index overload at the time it took to compile time. Starting
			from [3, 77, 4, 77, 5, 77, 6], this removes the 77 at index 5,
			leaving [3, 77, 4, 77, 5, 6].

		list.remove(Integer.valueOf(5)); // what does this one do?

			It Removes the first element whose values are equals 5. Integer.valueOf(5)
			produces an Integer object rather than an int, so overload resolution
			selects remove(Object o), which searches for a matching element.
			Starting from [3, 77, 4, 77, 5, 6], this removes the 5, leaving
			[3, 77, 4, 77, 6].

TestIterator.java

	testRemove()

		i.remove(); // what happens if you use list.remove(77)?

			Calling list.remove(...) directly inside the iterator loop throws a
			ConcurrentModificationException. The iterator keeps an internal
			modification counter that it compares against the list's on each call
			to next(); modifying the list through the list's own methods changes
			the list's counter without updating the iterator's, so the mismatch is
			detected and the exception is thrown. The iterator's own remove()
			method exists precisely to avoid this - it removes the element and
			keeps the two counters in sync. (Note also that list.remove(77) would
			bind to the index overload, not the value overload.)

TestPerformance.java

	Each configuration was run 3 times with REPS = 1000000, varying only the
	SIZE constant. Running times were read from the Gradle HTML test report at
	build/reports/tests/test/index.html, which lists a per-method duration in
	seconds; values below are converted to milliseconds. The first run at each
	size is consistently the slowest, reflecting JIT warmup, which is why
	multiple runs were recorded.

	SIZE 10
	                           #1    #2    #3
	testArrayListAddRemove:   451   454   436
	testLinkedListAddRemove:   53    32    34
	testArrayListAccess:       38    40    33
	testLinkedListAccess:      28    18    18

	SIZE 100
	                           #1    #2    #3
	testArrayListAddRemove:   537   448   478
	testLinkedListAddRemove:   41    43    31
	testArrayListAccess:       39    37    35
	testLinkedListAccess:      38    39    52

	SIZE 1000
	                           #1    #2    #3
	testArrayListAddRemove:   646   654   679
	testLinkedListAddRemove:   33    29    40
	testArrayListAccess:       48    40    37
	testLinkedListAccess:     482   446   459

	SIZE 10000
	                           #1    #2    #3
	testArrayListAddRemove:  4921  2072  1727
	testLinkedListAddRemove:   46    36    40
	testArrayListAccess:       43    39    40
	testLinkedListAccess:    8139  4978  5425

	listAccess - which type of List is better to use, and why?

		ArrayList, decisively. Its access time stayed essentially constant at
		roughly 35-45 ms across every size tested, because an ArrayList is backed
		by an array and get(i) computes an offset directly - constant time
		regardless of how large the list is.

		LinkedList access degraded in proportion to SIZE: about 20 ms at SIZE 10,
		40 ms at 100, 460 ms at 1000, and 6200 ms at 10000. Each get(i) must
		traverse the chain of nodes from one end to reach position i, so the cost
		per lookup grows linearly with the list length. At SIZE 10000 LinkedList
		was roughly 150x slower than ArrayList for the same million lookups.

	listAddRemove - which type of List is better to use, and why?

		LinkedList, and the margin widens as the list grows. Its add/remove time
		held flat at roughly 30-50 ms at every size, because inserting or removing
		at index 0 only requires updating the head node's links - the rest of the
		list is untouched.

		ArrayList add/remove at index 0 costs about 450 ms at SIZE 10 and rises to
		roughly 1700-4900 ms at SIZE 10000. Every insertion at the front must shift
		all existing elements one position right, and every removal shifts them
		back left, so the work per operation grows with the list length.

		The overall conclusion is that neither structure is universally better.
		ArrayList wins on indexed access and LinkedList wins on insertion and
		removal near the front, and the correct choice depends on which operation
		a program performs most often.