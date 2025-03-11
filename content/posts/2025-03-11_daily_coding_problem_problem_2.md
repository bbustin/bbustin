+++
title = "Daily Coding Problem: Problem #2"

[taxonomies]
tags = ["CodingInterview"]
+++

This is
[Daily Coding Problem](https://www.dailycodingproblem.com/) #2. I did the work in a Jupyter
notebook and then exported it to include here.
<!--more-->

Note: The color output of the grader method does work, but it did not export properly.

---

# Daily Coding Problem
## Problem #2 (Hard)

> This problem was asked by Uber.

> Given an array of integers, return a new array such that each element at index i of the new array is the product of all the numbers in the original array except the one at i.

> For example, if our input was [1, 2, 3, 4, 5], the expected output would be [120, 60, 40, 30, 24]. If our input was [3, 2, 1], the expected output would be [2, 3, 6].

> Follow-up: what if you can't use division?

# My Thoughts

We're back to math, not my strong suit. If I remember correctly, product is the result of multiplying two numbers.

To make sure I am understanding the problem correctly, let me manually work through the problem.


```python
# given an input of [1, 2, 3, 4, 5] - expected output of [120, 60, 40, 30, 24]

[
    2*3*4*5,
    3*4*5,
    4*5
]
```




    [120, 60, 20]



My initial understanding of the problem is not correct. It is not just advancing down the line. Otherwise the result would always have two less than the input.

Having that 1 at the beginning also threw me off a little because the second number in my result was
correct. If you look at the third number, it needed to be multiplied by 2 as well.

So, basically, we are just excluding the one number we are on.

The follow-up question mentions trying to do it without division. Had I not had that clue, I would not have realized I could just multiply all numbers and then divide by the number I am currently on.

Wait a sec, let me validate that assumption...


```python
# given an input of [1, 2, 3, 4, 5] - expected output of [120, 60, 40, 30, 24]

p_all = 1*2*3*4*5

[
    p_all/1,
    p_all/2,
    p_all/3,
    p_all/4,
    p_all/5
]

```




    [120.0, 60.0, 40.0, 30.0, 24.0]



That does work, for this example...

I think it might fail in certain scenarios. What if one value was a 0?


```python
p_all = 0*1*2*3*4

[
    p_all/0,
    p_all/1,
    p_all/2,
    p_all/3,
    p_all/4
]
```


    ---------------------------------------------------------------------------

    ZeroDivisionError                         Traceback (most recent call last)

    Cell In[3], line 4
          1 p_all = 0*1*2*3*4
          3 [
    ----> 4     p_all/0,
          5     p_all/1,
          6     p_all/2,
          7     p_all/3,
          8     p_all/4
          9 ]


    ZeroDivisionError: division by zero


It does not work in that case because you can't divide by zero. Also the product of all of the numbers is 0, but the first product returned should not be 0, but rather 24!

I think this will also fail if some of the numbers are negative.

Let's try.


```python
p_all = -1*2*3*4*5

[
    p_all/-1,
    p_all/2,
    p_all/3,
    p_all/4,
    p_all/5
]
```


```python
p_all = -1*2*-3*4*5

[
    p_all/-1,
    p_all/2,
    p_all/-3,
    p_all/4,
    p_all/5
]
```

Negative numbers do not break this. So, here are some of the edge cases identified so far which need to be in our testing.

* The number 0 is included
* One or more numbers is negative
* The list only has 1 number
* The list has no numbers

# Game plan

* Create some test cases and grading code
* Create a naïve implementation that is definitely not optimized, but hopefully will not fall victim to the 0 and negative number issue identified from using division, which I only tried beccause of the follow-up in the question.
* Maybe see if I can create a division implementation that does not fall victim to these issues
* See if I casn think of anything more efficient
* Use AI to see how close or far I got from the optimal answer (I'm not doing any research or using external tools until this - otherwise this will not really help prepare me for a real coding interview - ick)

# Test cases and grading code


```python
test_cases = [
    {
        'numbers': [1, 2, 3, 4, 5],
        'correct_answer': [120, 60, 40, 30, 24]
    },
    {
        'numbers': [0, 1, 2, 3, 4],
        'correct_answer': [24, 0, 0, 0, 0]
    },
    {
        'numbers': [-1, 2, 3, 4, 5],
        'correct_answer': [120, -60, -40, -30, -24]
    },
    {
        'numbers': [1],
        'correct_answer': [0] #I'm assuming 0 is the correct answer here?
    },
    {
        'numbers': [],
        'correct_answer': [] #we always return a list the same size as the input list
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
        numbers = test_case['numbers']
        correct_answer = test_case['correct_answer']

        result = code_under_test(numbers)
        test_passed = result == correct_answer

        # so we can print at the end that not all tests succeeded
        if not test_passed: test_failed = True

        test_result = "PASSED" if test_passed else "FAILED"
        line_color = GREEN if test_passed else RED
        if show_passed or not test_passed:
            print(f"{line_color}Test Case: numbers={numbers}, correct_answer={correct_answer} returned {result} - {test_result}{RESET}")

    execution_time = time.time() - start_time
    print(f"\nThis answer {RED + 'did not pass' if test_failed else GREEN + 'passed'}{RESET} all {len(test_cases)} tests. Ran for {execution_time:.8f} seconds.")
    return {'passed': test_passed, 'time': execution_time}
```

# Naïve implementation
For this, I'm just going to perform the mutiplication for each step. I will need to exclude the current number from the list.

I'm not going to try anything fancy here or even to use great data structures...


```python
def first_try(numbers):
    result = []
    # I bet there is a much more 'pythonic' way. I'm just coming back to Python after years of Java on the job
    for index in range(len(numbers)):
        # I think there is a way to use slices here instead of another loop, but bear with me...
        product = 1 #set to 1 so we can just multiply it - this might break for some edge cases - we will see
        for inner_index in range(len(numbers)):
            # the current number we are on should be excluded from the product
            if inner_index == index:
                break;
            product *= numbers[inner_index]
        result.append(product)

grade(first_try)
```

That's embarrassing... I did not even return anything. :facepalm:


```python
def second_try(numbers):
    result = []
    # I bet there is a much more 'pythonic' way. I'm just coming back to Python after years of Java on the job
    for index in range(len(numbers)):
        # I think there is a way to use slices here instead of another loop, but bear with me...
        product = 1 #set to 1 so we can just multiply it - this might break for some edge cases - we will see
        for inner_index in range(len(numbers)):
            # the current number we are on should be excluded from the product
            if inner_index == index:
                break;
            product *= numbers[inner_index]
        result.append(product)
    return result

grade(second_try)
```

I am not off to a great start here. Notice that only the last number is the only one that is right. The only test case that passed was `[]`.

I can easily fix the edge case with just one number by just detecting that at the beginning. I don't need to calculate the length twice, even though that is not really a big performance issue.

The main algorithm; however, is flawed. I need to figure out what is wrong. Sure, I could go through this and pretend I am a computer, but since I have a computer, let's just use trusty old print...


```python
def second_try(numbers):
    print("="*10)
    numbers_length = len(numbers)
    print(f"There are {numbers_length} numbers")

    # edge case - only 1 number should return 0
    if numbers_length == 0:
        return [0]

    result = []
    # I bet there is a much more 'pythonic' way. I'm just coming back to Python after years of Java on the job
    for index in range(len(numbers)):
        print(f"Working on index {index} which corresponds to number {numbers[index]}")
        # I think there is a way to use slices here instead of another loop, but bear with me...
        product = 1 #set to 1 so we can just multiply it - this might break for some edge cases - we will see
        for inner_index in range(len(numbers)):
            print(f"- inner index {inner_index} which corresponds to number {numbers[inner_index]}")
            # the current number we are on should be excluded from the product
            if inner_index == index:
                print("-- ignoring")
                break;
            product *= numbers[inner_index]
            print(f"-- product: {product}")
        result.append(product)
        print(f"> result so far: {result}")
    return result

grade(second_try)
```

Ok, color me embarrassed once more. The problem was the `break`. It should have been `continue`. I was just dumping out of the inner loop.


```python
def second_try(numbers):
    numbers_length = len(numbers)

    # edge case - only 1 number should return 0
    if numbers_length == 0:
        return [0]

    result = []
    # I bet there is a much more 'pythonic' way. I'm just coming back to Python after years of Java on the job
    for index in range(len(numbers)):
        # I think there is a way to use slices here instead of another loop, but bear with me...
        product = 1 #set to 1 so we can just multiply it - this might break for some edge cases - we will see
        for inner_index in range(len(numbers)):
            # the current number we are on should be excluded from the product
            if inner_index == index:
                continue;
            product *= numbers[inner_index]
        result.append(product)
    return result

grade(second_try)
```

Much better. I still flubbed the two edge cases, but at least the other tests are working. Huzzah. Look at that, I was testing for `numbers_length == 0` when I needed to test for `if numbers_length == 1`. Even the comment indicated that.


```python
def second_try(numbers):
    numbers_length = len(numbers)

    # edge case - only 1 number should return 0
    if numbers_length == 1:
        return [0]

    result = []
    # I bet there is a much more 'pythonic' way. I'm just coming back to Python after years of Java on the job
    for index in range(len(numbers)):
        # I think there is a way to use slices here instead of another loop, but bear with me...
        product = 1 #set to 1 so we can just multiply it - this might break for some edge cases - we will see
        for inner_index in range(len(numbers)):
            # the current number we are on should be excluded from the product
            if inner_index == index:
                continue;
            product *= numbers[inner_index]
        result.append(product)
    return result

grade(second_try)
```

Much better. My first passing one. Does this count as having done the challenge? I did not use division.

# Division implementation which does not fall victim to the edge cases

To do this, I am going to need to look out for a few things.

1. If the number is a 0, will need to get the product of all other numbers.
2. If there is only 1 number, return [0]
3. If there are no numbers, return []

Pseudo-code

* Get length of input numbers
* Check for 3 and 4, return the appropriate reponse if either condition
* Get the product of all numbers
* Go through each number, divide the product of all numbers by this number


```python
import math

def division_answer(numbers):
    numbers_length = len(numbers)

    # two edge cases
    if numbers_length == 0:
        return []
    elif numbers_length == 1:
        return [0]

    p_all = math.prod(numbers)

    result = []
    for number in numbers:
        product = p_all/number
        result.append(product)

    return result

grade(division_answer)
```

Argh! Forgot about the divide by zero edge case. What do I need to do there? Let me look at an example from the test cases with a 0...

`[0, 1, 2, 3, 4]` which should return `[24, 0, 0, 0, 0]`. So, essentially, if the number we are currently working on is 0 then there is a nonzero product (as long as there are no other 0).

The product of all is going to be 0, so, in this kind of scenario I am assuming we need just need the product of every other number minus this one. It makes this division route seem not as nice; however, there would still be fewer mathematical operations, I guess.

But now we need to keep the index too. I think there is a construct in Python to get the index and item without having to do alength call and using the range. That would be so much cleaner. I am going to allow myself to `cheat` on this aspect and research it.


```python
import math

def division_answer(numbers):
    numbers_length = len(numbers)

    # two edge cases
    if numbers_length == 0:
        return []
    elif numbers_length == 1:
        return [0]

    p_all = math.prod(numbers)

    result = []
    for index, number in enumerate(numbers):
        # edge case of a 0
        if number == 0:
            # ok, I 'cheated' on this one as well. I did not want to create another loop in here. It would make the code way too crappy.
            result.append(math.prod([value for i, value in enumerate(numbers) if i != index]))
        else:
            product = p_all/number
            result.append(product)

    return result

grade(division_answer)
```

All tests are now passing for the division way of doing it as well.

# What does Qwen2.5-coder think?

>
    I was given the following coding question:
    ... '
    ...
    ...     Given an array of integers, return a new array such that each element at index i of the new array is the p
    ... roduct of all the numbers in the original array except the one at i.
    ...
    ...     For example, if our input was [1, 2, 3, 4, 5], the expected output would be [120, 60, 40, 30, 24]. If our
    ... input was [3, 2, 1], the expected output would be [2, 3, 6].
    ...
    ...     Follow-up: what if you can't use division?
    ... '
    ...
    ... I answered in two ways. The first was without using division:
    ... ```python
    ... def second_try(numbers):
    ...     numbers_length = len(numbers)
    ...
    ...     # edge case - only 1 number should return 0
    ...     if numbers_length == 1:
    ...         return [0]
    ...
    ...     result = []
    ...     # I bet there is a much more 'pythonic' way. I'm just coming back to Python after years of Java on the job
    ...
    ...     for index in range(len(numbers)):
    ...         # I think there is a way to use slices here instead of another loop, but bear with me...
    ...         product = 1 #set to 1 so we can just multiply it - this might break for some edge cases - we will see
    ...         for inner_index in range(len(numbers)):
    ...             # the current number we are on should be excluded from the product
    ...             if inner_index == index:
    ...                 continue;
    ...             product *= numbers[inner_index]
    ...         result.append(product)
    ...     return result```
    ...
    ... In my second answer, I used division:
    ... ```python
    ... def division_answer(numbers):
    ...     numbers_length = len(numbers)
    ...
    ...     # two edge cases
    ...     if numbers_length == 0:
    ...         return []
    ...     elif numbers_length == 1:
    ...         return [0]
    ...
    ...     p_all = math.prod(numbers)
    ...
    ...     result = []
    ...     for index, number in enumerate(numbers):
    ...         # edge case of a 0
    ...         if number == 0:
    ...             # ok, I 'cheated'on this one as well. I did not want to create another loop in here. It would make
    ...  the code way too crappy.
    ...             result.append(math.prod([value for i, value in enumerate(numbers) if i != index]))
    ...         else:
    ...             product = p_all/number
    ...             result.append(product)
    ...
    ...     return result```
    ...
    ... Can you analyze these two responses for big O. Then please comment on them.  Finally, please suggest the most
    ... optimal way to solve the question.
    ... """

Certainly! Let's analyze your solutions in terms of time complexity and then discuss their efficiency and
readability. We'll also explore a more optimal solution.

### Analysis of Your Solutions

#### First Solution: Without Division

```python
def second_try(numbers):
    numbers_length = len(numbers)

    # edge case - only 1 number should return 0
    if numbers_length == 1:
        return [0]

    result = []
    for index in range(len(numbers)):
        product = 1
        for inner_index in range(len(numbers)):
            if inner_index == index:
                continue;
            product *= numbers[inner_index]
        result.append(product)
    return result
```

**Time Complexity:**
- The outer loop runs `n` times, where `n` is the length of the input list.
- The inner loop also runs `n` times for each iteration of the outer loop.
- Therefore, the overall time complexity is \(O(n^2)\).

**Space Complexity:**
- The space complexity is \(O(1)\) extra space (excluding the output list), as we are only using a few
variables.

**Comments:**
- This solution works correctly but is inefficient for large lists due to its quadratic time complexity.
- It is straightforward and easy to understand, which is good for educational purposes.

#### Second Solution: With Division

```python
import math

def division_answer(numbers):
    numbers_length = len(numbers)

    # two edge cases
    if numbers_length == 0:
        return []
    elif numbers_length == 1:
        return [0]

    p_all = math.prod(numbers)

    result = []
    for index, number in enumerate(numbers):
        # edge case of a 0
        if number == 0:
            result.append(math.prod([value for i, value in enumerate(numbers) if i != index]))
        else:
            product = p_all / number
            result.append(product)

    return result
```

**Time Complexity:**
- Calculating the total product `p_all` takes \(O(n)\).
- For each element, if it is zero, calculating the product of the rest of the elements takes \(O(n)\). This
means in the worst case (when there are multiple zeros), this could be inefficient.
- Therefore, the overall time complexity is \(O(n^2)\) in the worst case due to handling multiple zeros.

**Space Complexity:**
- The space complexity is \(O(1)\) extra space (excluding the output list).

**Comments:**
- This solution is more efficient than nested loops when there are no zeros, but it becomes inefficient with
zeros.
- It uses division, which is not allowed by the follow-up constraint.

### Optimal Solution Without Division

To solve this problem without using division and achieve a better time complexity, we can use prefix and
suffix products. Here's how:

1. **Prefix Product:** Calculate the product of all elements to the left of each element.
2. **Suffix Product:** Calculate the product of all elements to the right of each element.
3. **Result Calculation:** Multiply the corresponding prefix and suffix products for each element.

This approach ensures that we only traverse the list a few times, resulting in linear time complexity.

```python
def optimal_solution(numbers):
    numbers_length = len(numbers)

    if numbers_length == 0:
        return []
    elif numbers_length == 1:
        return [0]

    # Initialize prefix and suffix arrays
    prefix_products = [1] * numbers_length
    suffix_products = [1] * numbers_length

    # Calculate prefix products
    for i in range(1, numbers_length):
        prefix_products[i] = prefix_products[i - 1] * numbers[i - 1]

    # Calculate suffix products
    for i in range(numbers_length - 2, -1, -1):
        suffix_products[i] = suffix_products[i + 1] * numbers[i + 1]

    # Calculate the result by multiplying prefix and suffix products
    result = [prefix_products[i] * suffix_products[i] for i in range(numbers_length)]

    return result

# Example usage:
print(optimal_solution([1, 2, 3, 4, 5]))  # Output: [120, 60, 40, 30, 24]
print(optimal_solution([3, 2, 1]))        # Output: [2, 3, 6]
```

**Time Complexity:**
- Calculating prefix products takes \(O(n)\).
- Calculating suffix products takes \(O(n)\).
- Constructing the result array takes \(O(n)\).
- Therefore, the overall time complexity is \(O(n)\).

**Space Complexity:**
- The space complexity is \(O(n)\) due to the additional prefix and suffix arrays.

### Comments on Optimal Solution

- This solution is efficient with a linear time complexity.
- It avoids using division, adhering to the follow-up constraint.
- While it uses extra space for prefix and suffix products, this can be further optimized by modifying the
input array in place or using a single result array to store intermediate results.

### Final Thoughts

Your initial solutions were correct but not optimal. The optimal solution using prefix and suffix products
is both efficient and adheres to the constraints provided in the problem statement.

# My conclusion
I would have never figured out the O(n) solution. I still need to step through it and understand it better, but I need to head out shortly.
