# Exceptions and exception handling

Exceptions are a way to deal with abnormal situations in code. For example, consider the `pop` function of a `queue`. What should a `pop` from an empty queue do?

1. One answer is to say: It should have an undefined behavior and whoever calls us better be more careful about it and not try to call `pop` on an empty queue. After all, one of the *preconditions* for `pop` is that the queue is non-empty.
2. A better approach is to use an exception, which is effectively a *signal* to the caller that something wrong is going on. **An exception is like an abnormal return from a function.**
3. A third approach, worth knowing but not pertinent to our current discussion, is to return some sort of "impossible value", for example returning `-1` for the age of a person. This has its own problems though.

There are two important modes of operation when using exceptions:

- A function may **throw an exception** to signal the abnormal termination of its work. For example the `pop` function could *throw* a `domain_error` exception, one of a bunch of built-in exception types, to signal that we tried to call this function in a situation that doesn't apply to it. The code for this would look like this: 

    ```cpp
    value_type queue::pop()
    {
      if (empty()) throw domain_error("You tried to pop from an empty queue, silly programmer!");
      .... continue with pop steps ....
    }
    ```
- A function may **catch an exception** from a piece of code, then do something in response. This will prevent the program from abruptly terminating. We do this in a so-called *try-catch block*:

    ```cpp
    try
    {
      .... do some stuff here ....
      .... this stuff may throw a domain_error exception ....
    }
    catch (domain_error& e)
    {
      .... do something graceful in response to the exception ....
      .... e.what() contains the message ....
    }
    ```

### An example

As an example, let's write a function that is given a queue of hungry kittens. If the queue is non-empty, then we pop the kitten at the top and return it. If the queue is empty, we throw a `domain_error` exception.
```cpp
// Assuming there is a cat class
cat getNextHungryKitten(std::queue<cat> hungryKittens)
{
  if (hugryKittens.empty()) throw domain_error("There were no hungry kittens!");
  cat nextKitten = hungryKittens.top();
  hungryKittens.pop();
  return nextKitten;
}
```
And here we use this function in another function:
```cpp
void feedNextKitten(std::queue<cat> hungryKittens)
{
  try
  {
    cat nextKitten = getNextHungryKitten(hungryKittens);
    cout << "We are feeding " << nextKitten;
  }
  catch (domain_error& e)
  {
    cout << "Yey! " << e.what();    // Prints: Yey! There were no hungry kittens!
  }
}
```

### A more complex example

One of the most important features of exceptions is that they don't necessarily relate a function and its caller. When an exception occurs, it might be caught at a level "higher" than the caller, it could for example be the caller's caller's caller's caller (also known, of course, as the great-great-grand-caller) that catches it. While we won't do such an extreme example, here's one case where this might come in handy.

Let's continue on our feeding-the-kittens theme. We could imagine a `main` function whose goal it is to feed the kittens. It may get a list of hungry kittens from some helper function, then it might obtain some "food" from another function, then it gives both the food and the list of kittens to a third function to feed them:
```cpp
int main() 
{
    food fd = getFood();
    deque<cat> hungryKittens = getHungryKittens();
    feedTheKittens(fd, hungryKittens);
}
```
Now let's imagine what our `feedTheKittens` function might do: It might actually call another function to help it:
```cpp
void feedTheKittens(food fd, deque<cat> kittens)
{
    for (auto iter = kittens.begin(); iter != kittens.end(); ++iter)
    {
        feedTheKitten(fd, *iter);
    }
}
```
Now `feedTheKitten` is the function that really knows how to feed a kitten, place the food in a bowl etc. But what should it do if there is no more food left? It must signal its callers that it was *not* able to do its job, so they can take appropriate action. Probably someone needs to go to the store and buy more food. But that sounds like a job for `main`. So we have a situation where information from `feedTheKitten` needs to effectively skip the `feedTheKittens` function that called it and go to the `main` function that called `feedTheKittens`. Exceptions are excellent for a situation like this:

1. It is an abnormal event that needs to be communicated somewhere higher-up.
2. The immediate caller of the function is not the appropriate place to handle the event.

So here is how `feedTheKitten` might look like:
```cpp
void feedTheKitten(food fd, cat kitten)
{
    if (fd.empty()) throw out_of_food();  // out_of_food is our own exception class 
    bowl b(fd);      // make a bowl using the food
    kitten.each(b);  // tell the kitten to eat from the bowl
}
```
Now we should change our `main` function to deal with this problem:
```cpp
int main() 
{
    food fd = getFood();
    deque<cat> hungryKittens = getHungryKittens();
    try 
    {
        feedTheKittens(fd, hungryKittens);
    }
    catch (out_of_food e)
    {
        // order more food
        // write a message that not all kittens were fed
        // ...
    }
}
```

This is the key takeaway from exceptions: **Exceptions allow some part of the code to deal with an abnormal event in a completely different part of the code."
