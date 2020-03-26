# Exceptions and exception handling

Exceptions are a way to deal with abnormal situations when code is executing. For example, consider the `pop` function of a `queue`. What should happen if we try to `pop` from an empty queue?

1. One possibility: The behavior is undefined and whoever made the call should be more careful and not try to call `pop` on an empty queue. After all, one of the *preconditions* for `pop` is that the queue is non-empty.
2. A better approach is to use an exception, which is effectively a *signal* to the caller that something wrong is going on. **An exception is like an abnormal return from a function.**
3. A third approach, worth knowing but not pertinent to our current discussion, is to return some sort of "impossible value", for example returning `-1` for the age of a person. This has its own problems though.

There are two important modes of operation when using exceptions:

- A function may **throw an exception** to signal the abnormal termination of its work. For example the `pop` function could *throw* a `domain_error` exception, one of the built-in exception types, to signal that the function was called in a situation that wasn't appropriate. The code for this would look like this: 

    ```cpp
    value_type queue::pop()
    {
      if (empty()) throw domain_error("You tried to pop from an empty queue, silly programmer!");
      .... continue with pop steps ....
    }
    ```
- A function may **catch an exception** from a piece of code, then do something in response. This will prevent the program from abruptly terminating. We do this using a *try-catch block*:

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

As an example, let's write a function that is given a queue of hungry kittens. If the queue is not empty, then we pop the kitten at the top and return it. If the queue is empty, we throw a `domain_error` exception.
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
Now let's use this function in another function:
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

One of the most important features of exceptions is that *the handler of a thrown exception does not have to be the caller of the function that threw the exception*. When an exception occurs, it may be caught at a level "higher" than the direct caller, it could for example be the caller's caller's caller's caller (also known, of course, as the great-great-grand-caller) that catches it. Note that this behavior of exceptions is very different from the "returning from a function" behavior, which always returns control to the function's caller.

Below is one situation where we want this behavior: The caller of the function is not the right place to handle the thrown exception.

Continuing the feeding-the-kittens theme, we could imagine a `main` function whose goal it is to feed the kittens. It may get a list of hungry kittens from some helper function, then it might obtain some "food" from another function, then it gives both the food and the list of kittens to a third function to take care of the feeding:
```cpp
int main() 
{
    food kibble = getFood();
    deque<cat> hungryKittens = getHungryKittens();
    feedAllKittens(kibble, hungryKittens);
}
```
Now let's imagine what our `feedAllKittens` function might do: It might actually call another function to help it:
```cpp
void feedAllKittens(food kibble, deque<cat> kittens)
{
    for (auto iter = kittens.begin(); iter != kittens.end(); ++iter)
    {
        feedTheKitten(kibble, *iter);
    }
}
```
Now `feedTheKitten` is the function that really knows how to feed a kitten, place the food in a bowl etc. But what should it do if there is no more food left? It must signal its callers that it was *not* able to do its job, so they can take appropriate action. Probably someone needs to go to the store and buy more food. But that sounds like a job for `main`. So we have a situation where information from `feedTheKitten` needs to effectively skip the `feedAllKittens` function that called it and go to the `main` function that called `feedAllKittens`. Exceptions are excellent for a situation like this because:

1. It is an abnormal event that needs to be communicated/handled somewhere higher-up.
2. The immediate caller of the function is not the appropriate place to handle the event.

So here is how `feedTheKitten` might look like:
```cpp
void feedTheKitten(food kibble, cat kitten)
{
    if (kibble.empty()) 
        throw out_of_food();  // out_of_food is our own exception class 
    bowl b(kibble);     // make a bowl using the food
    kitten.each(b);     // tell the kitten to eat from the bowl
}
```
Now we need to change our `main` function to deal with this problem:
```cpp
int main() 
{
    food kibble = getFood();
    deque<cat> hungryKittens = getHungryKittens();
    try 
    {
        feedAllKittens(kibble, hungryKittens);
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
