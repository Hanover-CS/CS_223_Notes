# Recursion 
## Recursive functions

- A recursive function can *call itself*
- Think of it like *making a clone and giving it a task similar to yours*
- You must *trust* that your clone will do the right thing
- Simple example: You are asked to compute 4+5+10
   - You make a clone of yourself and ask your clone to compute 5+10
   - Your clone goes off and does its work, and comes back with the answer of 15
   - Then you compute 4 + 15, which is 19, and you return that as your answer
   - Trust that your clone did the right thing
- High level logic for adding a bunch of numbers:
   - If there are no numbers, return 0
   - If there is at least one number:
      - put that one number aside, and
      - recursively call yourself on the remaining numbers
         - i.e. make a clone and give it the remaining numbers to add, then wait for your clone to return
      - then add your number to the answer you got from your clone

## Recursive structures

- A recursive structure contains a component that looks like a smaller version of the overall structure (typically with less data)
- Example: Linked list
- We can think of a linked list as consisting of a head node that contains two things:
  - A `data` value
  - A `rest` pointer to *another* linked list (namely all the remaining nodes)
  
      ```cpp
      class node 
      {
        public:
          int data;
          node* rest;
          node(int data, node* rest);
      }
      ```
- We can write recursive functions that operate on a linked list this way:
  - Call yourself/your clone on the `rest` of the list, and possibly get back a result
  - Do something with that result and with `data`
- Example, adding all the numbers in a linked list:

    ```cpp
    int add_up(node* node)
    {
      if (node == NULL) return 0;         // If at end of list, return
      int sum_rest = add_up(node->rest);  // Add up all the rest
      return data + sum_rest;             // Now add the data to that
    }

    // Or as a node member function:

    int add_up()    // the node is "this"
    {
      if (rest == NULL) return data;   // Note the difference!
      int sum_rest = rest->add_up();
      return data + sum_rest;  
    }
    ```
- Example 2, building a list:
   - We can recursively create a list of the numbers up to 5 as follows:
     - Create the list of the numbers up to 4
     - Add in front a new node for 5

   ```cpp
   node* make_list(int n)
   {
      if (n == 0) return NULL;
      node* rest = make_list(n - 1);
      return new node(n, rest);
   }
   ```
- Example use:
   
   ```cpp
   int main()
   {
      node* list = make_list(5);
      int sum = add_up(list);
      cout << sum;
   }
   ```
