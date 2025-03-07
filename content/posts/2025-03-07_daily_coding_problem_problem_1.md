+++
title = "Daily Coding Problem: Problem #1"

[taxonomies]
tags = ["CodingInterview"]
+++

One of my friends on LinkedIn recommended I sign up for the
[Daily Coding Problem](https://www.dailycodingproblem.com/) email. This is
Problem #1. I did the work in a Jupyter notebook and then exported it to include here.
<!--more-->

Note: The color output of the grader method does work, but it did not export properly.

---

# The problem
(Daily Coding Problem: Problem #1 [Easy])

This problem was recently asked by Google.

Given a list of numbers and a number k, return whether any two numbers from the list add up to k.

For example, given [10, 15, 3, 7] and k of 17, return true since 10 + 7 is 17.

Bonus: Can you do this in one pass?

# My thoughts

* Addition is commutative, so we only need to try each combination once
* Should be able to easily do this in one pass by taking first number and adding it to each other number. Then slide forward and do the same. In this way we should not repeat any combinations.
* It looks like the ask is only to see if any two numbers add up to k, so there is no need to keep track of the combinations and we can exit out if any combinations are found.

## Possible edge cases

* Negative numbers, but this should still work fine as -1 and 10, or 10 and -1 should be the same. Let's test, but addition is commutative, so I'm pretty sure.


```python
print(f"-1 + 10 = { -1 + 10 }")
print(f"10 + -1 = { 10 + -1 }")
```

    -1 + 10 = 9
    10 + -1 = 9


# common setup code - test cases and grading code


```python
test_cases = [
    {
        'k': 10,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': 5,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': -1,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': 11,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': False
    },
]

# ANSI escape codes for color on the terminal
RED = "\033[91m"
GREEN = "\033[92m"
RESET = "\033[0m"

import time

def grade(code_under_test, show_passed=False):
    test_failed = False

    start_time = time.time()
    for test_case in test_cases:
        k = test_case['k']
        numbers = test_case['numbers']
        correct_answer = test_case['correct_answer']

        result = code_under_test(k, numbers)
        test_passed = result == correct_answer

        # so we can print at the end that not all tests succeeded
        if not test_passed: test_failed = True

        test_result = "PASSED" if test_passed else "FAILED"
        line_color = GREEN if test_passed else RED
        if show_passed or not test_passed:
            print(f"{line_color}Test Case: k={k}, numbers={numbers}, correct_answer={correct_answer} returned {result} - {test_result}{RESET}")

    execution_time = time.time() - start_time
    print(f"\nThis answer {RED + 'did not pass' if test_failed else GREEN + 'passed'}{RESET} all {len(test_cases)} tests. Ran for {execution_time:.8f} seconds.")
    return {'passed': test_passed, 'time': execution_time}
```


```python
# First try

def first_try(k, numbers):
    numbers_length = len(numbers)

    # range is start_inclusive, but stop exclusive
    for left_index in range(0, numbers_length):
        # if we are on the last number, there is nothing to do
        if (left_index + 1) == numbers_length:
            break

        for right_index in range(left_index + 1, numbers_length):
            left_number = numbers[left_index]
            right_number = numbers[right_index]

            if (left_number + right_number) == k:
                return True

    return False

grade(first_try, show_passed=True)

```

    [92mTest Case: k=10, numbers=[2, 8, 5, 0, -1, 5, 4], correct_answer=True returned True - PASSED[0m
    [92mTest Case: k=5, numbers=[2, 8, 5, 0, -1, 5, 4], correct_answer=True returned True - PASSED[0m
    [92mTest Case: k=-1, numbers=[2, 8, 5, 0, -1, 5, 4], correct_answer=True returned True - PASSED[0m
    [92mTest Case: k=11, numbers=[2, 8, 5, 0, -1, 5, 4], correct_answer=False returned False - PASSED[0m

    This answer [92mpassed[0m all 4 tests. Ran for 0.00003576 seconds.





    {'passed': True, 'time': 3.5762786865234375e-05}



I know this does not satisfy the one pass criteria as there are two loops. I queried qwen2.5-coder for assistance...

>
    ... Given the question:
    ...
    ... Given a list of numbers and a number k, return whether any two numbers from the list add up to k.
    ...
    ... For example, given [10, 15, 3, 7] and k of 17, return true since 10 + 7 is 17.
    ...
    ... Bonus: Can you do this in one pass?
    ...
    ... Is this a good answer? What is its o complexity and could it be accomplished more efficiently?
    ...
    ... def first_try(k, numbers):
    ...     numbers_length = len(numbers)
    ...
    ...     # range is start_inclusive, but stop exclusive
    ...     for left_index in range(0, numbers_length):
    ...         # if we are on the last number, there is nothing to do
    ...         if (left_index + 1) == numbers_length:
    ...             break
    ...
    ...         for right_index in range(left_index + 1, numbers_length):
    ...             left_number = numbers[left_index]
    ...             right_number = numbers[right_index]
    ...
    ...             if (left_number + right_number) == k:
    ...                 return True
    ...
    ...     return False

Your provided solution is a valid approach to solving the problem, but it can be improved in terms of time
complexity. Let's analyze your solution and then discuss more efficient alternatives.

### Analysis of Your Solution

**Code:**
```python
def first_try(k, numbers):
    numbers_length = len(numbers)

    # range is start_inclusive, but stop exclusive
    for left_index in range(0, numbers_length):
        # if we are on the last number, there is nothing to do
        if (left_index + 1) == numbers_length:
            break

        for right_index in range(left_index + 1, numbers_length):
            left_number = numbers[left_index]
            right_number = numbers[right_index]

            if (left_number + right_number) == k:
                return True

    return False
```

**Time Complexity:**
- The outer loop runs `n` times, where `n` is the length of the list.
- For each iteration of the outer loop, the inner loop runs on average `n/2` times (since it starts from
`left_index + 1`).
- Therefore, the overall time complexity is \(O(n^2)\).

**Space Complexity:**
- The space complexity is \(O(1)\) because you are not using any additional data structures that grow with
input size.

### More Efficient Solution

To solve this problem in one pass (i.e., \(O(n)\) time complexity), you can use a set to keep track of the
numbers you have seen so far. This allows you to check if the complement of the current number (i.e., `k -
current_number`) has already been encountered.

**Code:**
```python
def more_efficient(k, numbers):
    seen_numbers = set()

    for number in numbers:
        complement = k - number
        if complement in seen_numbers:
            return True
        seen_numbers.add(number)

    return False
```

**Time Complexity:**
- The loop runs `n` times, where `n` is the length of the list.
- Each operation (checking membership and adding to a set) on average takes \(O(1)\) time.
- Therefore, the overall time complexity is \(O(n)\).

**Space Complexity:**
- The space complexity is \(O(n)\) because in the worst case, you might store all `n` numbers in the set.

### Explanation of the Efficient Solution

1. **Initialization**: Start with an empty set `seen_numbers`.
2. **Iteration**: Loop through each number in the list.
3. **Complement Check**: For each number, calculate its complement (`k - number`). If this complement is
already in `seen_numbers`, return `True` because you have found two numbers that add up to `k`.
4. **Add to Set**: If the complement is not found, add the current number to `seen_numbers`.
5. **Completion**: If the loop completes without finding any such pair, return `False`.

### Conclusion

- Your original solution has a time complexity of \(O(n^2)\), which is less efficient for large lists.
- The more efficient solution using a set has a time complexity of \(O(n)\) and a space complexity of
\(O(n)\), making it suitable for larger inputs.

The efficient solution meets the bonus requirement of solving the problem in one pass.

# Let's try the AI answer and get a better understanding of it...


```python
def more_efficient(k, numbers):
    seen_numbers = set()

    for number in numbers:
        complement = k - number
        if complement in seen_numbers:
            return True
        seen_numbers.add(number)

    return False

grade(more_efficient, show_passed=True)
```

    [92mTest Case: k=10, numbers=[2, 8, 5, 0, -1, 5, 4], correct_answer=True returned True - PASSED[0m
    [92mTest Case: k=5, numbers=[2, 8, 5, 0, -1, 5, 4], correct_answer=True returned True - PASSED[0m
    [92mTest Case: k=-1, numbers=[2, 8, 5, 0, -1, 5, 4], correct_answer=True returned True - PASSED[0m
    [92mTest Case: k=11, numbers=[2, 8, 5, 0, -1, 5, 4], correct_answer=False returned False - PASSED[0m

    This answer [92mpassed[0m all 4 tests. Ran for 0.00003123 seconds.





    {'passed': True, 'time': 3.123283386230469e-05}



I asked the AI for more guidance as my math skills are not the greatest...

> What is a complement as it pertains to mathematics?

In mathematics, the term "complement" can have different meanings depending on the context. However, in the
specific context of the problem you're working on—checking if any two numbers from a list add up to a given
number \( k \)—the concept of a complement is used to identify the number that, when added to the current
number, equals \( k \).

### Complement in This Context

Given a number \( \text{number} \) and a target sum \( k \), the **complement** of \( \text{number} \) with
respect to \( k \) is:

\[ \text{complement} = k - \text{number} \]

This complement is the value that, when added to \( \text{number} \), results in \( k \).

### Example

Consider the list `[10, 15, 3, 7]` and the target sum \( k = 17 \):

- For `number = 10`, the complement is \( 17 - 10 = 7 \).
- For `number = 15`, the complement is \( 17 - 15 = 2 \).
- For `number = 3`, the complement is \( 17 - 3 = 14 \).
- For `number = 7`, the complement is \( 17 - 7 = 10 \).

In this example, when you check the number `10`, its complement is `7`. Since `7` is present in the list,
the function should return `True`.

### Efficient Solution Using Complements

Here's a step-by-step breakdown of how the efficient solution works using complements:

1. **Initialization**: Start with an empty set `seen_numbers`.
2. **Iteration**: Loop through each number in the list.
3. **Complement Check**: For each number, calculate its complement (`k - number`). If this complement is
already in `seen_numbers`, return `True` because you have found two numbers that add up to \( k \).
4. **Add to Set**: If the complement is not found, add the current number to `seen_numbers`.
5. **Completion**: If the loop completes without finding any such pair, return `False`.

### Code Example

```python
def more_efficient(k, numbers):
    seen_numbers = set()

    for number in numbers:
        complement = k - number
        if complement in seen_numbers:
            return True
        seen_numbers.add(number)

    return False
```

### Summary

- **Complement**: In this context, the complement of a number with respect to \( k \) is the value that,
when added to the number, equals \( k \).
- **Efficient Solution**: By using a set to track numbers you've seen so far and checking if the complement
of the current number exists in the set, you can solve the problem in one pass with a time complexity of \(
O(n) \).

This approach leverages the properties of sets for fast membership testing, making it much more efficient
than the nested loop method.

# More test cases
This now makes sense to me. :-)

I asked the AI to create some more test cases so I can retest, maybe see some of the performance difference,
as well as validate against addiitonal edge cases it identified.

From the AI:
### Explanation of Test Cases

1. **Basic Tests**:
   - Simple cases with positive and negative numbers.
   - Edge cases with zero and single-element lists.

2. **Edge Cases**:
   - Lists with no elements (`[]`).
   - Lists with a single element (`[0]`, `[5]`).

3. **Duplicates**:
   - Lists with duplicate numbers that can sum to `k` (`[5, 5]`, `[10, 10]`).

4. **Floating Point Numbers**:
   - Lists with floating-point numbers to ensure the function handles them correctly.

5. **Negative Numbers**:
   - Lists with negative numbers to test the function's ability to handle negative complements and sums.

6. **Various Sums**:
   - Multiple test cases with different target sums (`k`) to ensure the function works for various
scenarios.

These test cases should cover a wide range of possible inputs and edge cases, ensuring that the function is
robust and handles different types of input effectively.


```python
test_cases = [
    {
        'k': 10,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': 5,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': -1,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': 11,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': False
    },
    {
        'k': 17,
        'numbers': [10, 15, 3, 7],
        'correct_answer': True
    },
    {
        'k': 16,
        'numbers': [10, 15, 3, 7],
        'correct_answer': False
    },
    {
        'k': 0,
        'numbers': [0, 0],
        'correct_answer': True
    },
    {
        'k': 0,
        'numbers': [0],
        'correct_answer': False
    },
    {
        'k': 10,
        'numbers': [],
        'correct_answer': False
    },
    {
        'k': 5,
        'numbers': [5, 5],
        'correct_answer': True
    },
    {
        'k': 10,
        'numbers': [10],
        'correct_answer': False
    },
    {
        'k': 20,
        'numbers': [10, 10],
        'correct_answer': True
    },
    {
        'k': 3,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': 6,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': 7,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': 8,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': 9,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': False
    },
    {
        'k': -5,
        'numbers': [-10, 3, 7, -2, -8],
        'correct_answer': True
    },
    {
        'k': -15,
        'numbers': [-10, 3, 7, -2, -8],
        'correct_answer': False
    },
    {
        'k': 1,
        'numbers': [0.5, 0.5],
        'correct_answer': True
    },
    {
        'k': 1.5,
        'numbers': [0.5, 1, 2, 3],
        'correct_answer': True
    },
    {
        'k': 1.5,
        'numbers': [0.5, 0.5],
        'correct_answer': False
    },
]
```


```python
grade(first_try)
```

    [91mTest Case: k=5, numbers=[5, 5], correct_answer=True returned False - FAILED[0m
    [91mTest Case: k=9, numbers=[1, 2, 3, 4, 5], correct_answer=False returned True - FAILED[0m

    This answer [91mdid not pass[0m all 22 tests. Ran for 0.00008297 seconds.





    {'passed': True, 'time': 8.296966552734375e-05}



# first_try shows some failed test cases
> Test Case: k=5, numbers=[5, 5], correct_answer=True returned False - FAILED
  Test Case: k=9, numbers=[1, 2, 3, 4, 5], correct_answer=False returned True - FAILED

Since these test cases came from AI, and AI is not typically great at math yet, I need to dobule-check the test cases.

> Test Case: k=5, numbers=[5, 5], correct_answer=True returned False - FAILED

I think this one should really be False. The problem says add two numbers.

> Test Case: k=9, numbers=[1, 2, 3, 4, 5], correct_answer=False returned True - FAILED

4 + 5 = 9

I need to fix those two test cases.


```python
test_cases = [
    {
        'k': 10,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': 5,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': -1,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': True
    },
    {
        'k': 11,
        'numbers': [2, 8, 5, 0, -1, 5, 4],
        'correct_answer': False
    },
    {
        'k': 17,
        'numbers': [10, 15, 3, 7],
        'correct_answer': True
    },
    {
        'k': 16,
        'numbers': [10, 15, 3, 7],
        'correct_answer': False
    },
    {
        'k': 0,
        'numbers': [0, 0],
        'correct_answer': True
    },
    {
        'k': 0,
        'numbers': [0],
        'correct_answer': False
    },
    {
        'k': 10,
        'numbers': [],
        'correct_answer': False
    },
    {
        'k': 5,
        'numbers': [5, 5],
        'correct_answer': False
    },
    {
        'k': 10,
        'numbers': [10],
        'correct_answer': False
    },
    {
        'k': 20,
        'numbers': [10, 10],
        'correct_answer': True
    },
    {
        'k': 3,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': 6,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': 7,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': 8,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': 9,
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': True
    },
    {
        'k': -5,
        'numbers': [-10, 3, 7, -2, -8],
        'correct_answer': True
    },
    {
        'k': -15,
        'numbers': [-10, 3, 7, -2, -8],
        'correct_answer': False
    },
    {
        'k': 1,
        'numbers': [0.5, 0.5],
        'correct_answer': True
    },
    {
        'k': 1.5,
        'numbers': [0.5, 1, 2, 3],
        'correct_answer': True
    },
    {
        'k': 1.5,
        'numbers': [0.5, 0.5],
        'correct_answer': False
    },
]
```

# grade first_try again


```python
grade(first_try)
```


    This answer [92mpassed[0m all 22 tests. Ran for 0.00001407 seconds.





    {'passed': True, 'time': 1.4066696166992188e-05}



# grade more_efficient


```python
grade(more_efficient)
```


    This answer [92mpassed[0m all 22 tests. Ran for 0.00001192 seconds.





    {'passed': True, 'time': 1.1920928955078125e-05}



# Now, let's show the real performance difference between them!


```python
test_cases = []
for i in range(100_000_000):
    test_cases.append(
        {
            'k': 10,
            'numbers': [2, 8, 5, 0, -1, 5, 4],
            'correct_answer': True
        }
    )
```


```python
first_try_time = grade(first_try)['time']
```


    This answer [92mpassed[0m all 100000000 tests. Ran for 24.29532194 seconds.



```python
more_efficient_time = grade(more_efficient)['time']
```


    This answer [92mpassed[0m all 100000000 tests. Ran for 17.80483794 seconds.



```python
print(first_try_time - more_efficient_time)
```

    6.490483999252319


# Huge diffence
With 100 million records, the more efficient code finishes nearly 6 seconds faster. Not bad.
I need to remember the concept of using the complements for this.
