# k-th Largest Element Problem

You're given an integer array `a`. Write an algorithm that finds the *k*-th largest element in the array.

For example, the 1-st largest element is the maximum value that occurs in the array. If the array has *n* elements, the *k*-th largest element is the minimum. The median is the *k/2*-th largest element.

## The naive solution

The following solution is semi-naive. Its time complexity is **O(n log n)** since it first sorts the array, and therefore also uses additional **O(n)** space.

```go
import "sort"

func kthLargest(arr []int, k int) int {
  length := len(arr)
  
  if k > 0 && k <= length {
    sort.Ints(arr)
    return arr[length - k]
  }
  return -1
}
```

The `kthLargest()` function takes two parameters: the array `arr` consisting of integers, and `k`. It returns the *k*-th largest element.

Let's take a look at an example and run through the algorithm to see how it works. Given `k = 4` and the array:

```go
[ 7, 92, 23, 9, -1, 0, 11, 6 ]
```

Initially there's no direct way to find the k-th largest element, but after sorting the array it's rather straightforward. Here's the sorted array:

```go
[ -1, 0, 6, 7, 9, 11, 23, 92 ]
```

Now, all we must do is take the value at index `len(arr) - k`:

```go
arr[len(arr) - k] = arr[8 - 4] = arr[4] = 9
```

Of course, if you were looking for the k-th *smallest* element, you'd use `a[k]`.

## A faster solution

There is a clever algorithm that combines the ideas of [binary search](../Binary%20Search/) and [quicksort](../Quicksort/) to arrive at an **O(n)** solution.

Recall that binary search splits the array in half over and over again, to quickly narrow in on the value you're searching for. That's what we'll do here too.

Quicksort also splits up arrays. It uses partitioning to move all smaller values to the left of the pivot and all greater values to the right. After partitioning around a certain pivot, that pivot value will already be in its final, sorted position. We can use that to our advantage here.

Here's how it works: We choose a random pivot, partition the array around that pivot, then act like a binary search and only continue in the left or right partition. This repeats until we've found a pivot that happens to end up in the *k*-th position.

Let's look at the original example again. We're looking for the 4-th largest element in this array:

	[ 7, 92, 23, 9, -1, 0, 11, 6 ]

The algorithm is a bit easier to follow if we look for the n-th *smallest* item instead, so let's take `k = 4` and look for the 4-th smallest element.

Note that we don't have to sort the array first. We pick one of the elements at random to be the pivot, let's say `11`, and partition the array around that. We might end up with something like this:

	[ 7, 9, -1, 0, 6, 11, 92, 23 ]
	 <------ smaller    larger -->

As you can see, all values smaller than `11` are on the left; all values larger are on the right. The pivot value `11` is now in its final place. The index of the pivot is 5, so the 4-th smallest element must be in the left partition somewhere. We can ignore the rest of the array from now on:

	[ 7, 9, -1, 0, 6, x, x, x ]

Again let's pick a random pivot, let's say `6`, and partition the array around it. We might end up with something like this:

	[ -1, 0, 6, 9, 7, x, x, x ]

Pivot `6` ended up at index 2, so obviously the 4-th smallest item must be in the right partition. We can ignore the left partition:

	[ x, x, x, 9, 7, x, x, x ]

Again we pick a pivot value at random, let's say `9`, and partition the array:

	[ x, x, x, 7, 9, x, x, x ]

The index of pivot `9` is 4, and that's exactly the *k* we're looking for. We're done! Notice how this only took a few steps and we did not have to sort the array first.

The following function implements these ideas:

```go
func randomPivot(arr *[]int, min, max int) int {
	// choose a random pivot index
	pivotIndex := randomNumber(min, max)
	// swap the element at pivot <---> max position
	swap(arr, pivotIndex, max)
	return (*arr)[max]
}

func randomPartition(arr *[]int, min, max int) int {
	// fmt.Println("before random pivot")
	pivot := randomPivot(arr, min, max)
	// fmt.Println("after random pivot")

	i := min
	for j := min; j < max; j++ {
		if (*arr)[j] <= pivot {
			swap(arr, i, j)
			i = i + 1
		}
	}
	swap(arr, i, max)
	return i
}

func randomSelect(arr *[]int, min, max, k int) int {
	if min < max {
		p := randomPartition(arr, min, max)

		if k == p {
			return (*arr)[p]
		} else if k < p {
			return randomSelect(arr, min, p-1, k)
		} else {
			return randomSelect(arr, p+1, max, k)
		}

	} else {
		return (*arr)[min]
	}
}

}
```

To keep things readable, the functionality is split into three inner functions:

- `randomPivot()` picks a random number and puts it at the end of the current partition (this is a requirement of the Lomuto partitioning scheme, see the discussion on [quicksort](../Quicksort/) for more details).

- `randomPartition()` is Lomuto's partitioning scheme from quicksort. When this completes, the randomly chosen pivot is in its final sorted position in the array. It returns the array index of the pivot.

- `randomSelect()` does all the hard work. It first calls the partitioning function and then decides what to do next. If the index of the pivot is equal to the *k*-th number we're looking for, we're done. If `k` is less than the pivot index, it must be in the left partition and we'll recursively try again there. Likewise for when the *k*-th number must be in the right partition.

Pretty cool, huh? Normally quicksort is an **O(n log n)** algorithm, but because we only partition smaller and smaller slices of the array, the running time of `randomizedSelect()` works out to **O(n)**.


*Written by [brotherpowers](https://www.brotherpowers.com/)*