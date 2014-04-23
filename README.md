pqsort.h
========

Generic C99 partial quicksort.

This is the sort routine largely as described by Bentley & McIlroy's
"Engineering a Sort Function", along with some modifications as found in
FreeBSD's libc.

"Partial" means pqsort can be used to place only specific items in their final
sorted positions, making it useful for pagination, ranking and partitioning.

offset=100, limit=100 will place array[100] - array[199] in the correct
position, ensure array[0] - array[99] contains only values smaller than that in
array[100], and array[200] - array[nitems - 1] only values larger than
array[199].

Since it's a preprocessor macro, comparison functions can be inlined. This can
be quite a win if comparisons are cheap compared with function calls, e.g. when
sorting integers. It can be easily adapted to handle void *'s and support a
runtime-specified comparison function.'

A modified version of this routine was used by Newzbin.com's custom search
engine, quickly sorting upwards of 40 million items for result sets between 50
and 20,000.


Example
-------

	#include <stdio.h>
	#include "pqsort.h"

	static inline int IntCmp(const int a, const int b) {
		return (a - b);
	}

	define_pqsort(my_int, int, IntCmp);

	#define DUMP()  printf("{ "); \
		for (int i = 0; i < nvalues; i++)  \
			printf("%d, ", values[i]); \
		printf("}\n");

	int main(void) {
		int values[] = { 42, 98, 56, 23, 45, 63, 56, 80, 102, 2 };
		int nvalues = sizeof(values) / sizeof(values[0]);

		// partition the list in two
		my_int_pqsort(values, nvalues, 5, 0);
		DUMP(); // => { 23, 2, 42, 45, 56, 80, 102, 98, 63, 56, }

		// sort the first half
		my_int_pqsort(values, nvalues, 0, 5);
		DUMP(); // => { 2, 23, 42, 45, 56, 102, 98, 63, 80, 56, }

		// sort the rest
		my_int_pqsort(values + 5, nvalues - 5, 0, nvalues);
		DUMP(); // => { 2, 23, 42, 45, 56, 56, 63, 80, 98, 102, }

		return 0;
	}

