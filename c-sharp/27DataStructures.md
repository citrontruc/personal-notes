# Linked List

## Table of Content

- [Linked List](#linked-list)
  - [Table of Content](#table-of-content)
  - [Adding nodes](#adding-nodes)
  - [Doubly Linked List](#doubly-linked-list)
  - [Sorted List](#sorted-list)
  - [Deque](#deque)
  - [Binary search tree / SortedDictionary](#binary-search-tree--sorteddictionary)
    - [Traversal](#traversal)
    - [deleting a node with two children](#deleting-a-node-with-two-children)

## Adding nodes

If we want to have the last element of the list, we can have first and last values. These methods should be recursive ==> Careful.

## Doubly Linked List

Same as linked list but you can walk through the list in both directions: we must have next and previous methods to get the next node and the previous one. We can also have Head and Tail nodes to go directly to the first or last nodes. If you do that, you need two methods: AddHead and AddTail.

These nodes should have a { get; private set; } in order to avoid strange behaviours.

```cs
using System.Collections;

namespace DataStructures
{
    /// <summary>
    /// A node in the LinkedList
    /// </summary>
    /// <typeparam name="T"></typeparam>
    public class DoublyLinkedListNode<T>
    {
        /// <summary>
        /// Constructs a new node with the specified value.
        /// </summary>
        /// <param name="value"></param>
        public DoublyLinkedListNode(T value)
        {
            Value = value;
        }

        /// <summary>
        /// The node value
        /// </summary>
        public T Value { get; set; }

        /// <summary>
        /// The next node in the linked list (null if last node)
        /// </summary>
        public DoublyLinkedListNode<T> Next { get; set; }

        /// <summary>
        /// The previous node in the linked list (null if first node)
        /// </summary>
        public DoublyLinkedListNode<T> Previous { get; set; }
    }

    /// <summary>
    /// A linked list collection capable of basic operations such as 
    /// Add, Remove, Find and Enumerate
    /// </summary>
    /// <typeparam name="T"></typeparam>
    public class DoublyLinkedList<T> : 
        System.Collections.Generic.ICollection<T>
    {
        /// <summary>
        /// The first node in the list or null if empty
        /// </summary>
        public DoublyLinkedListNode<T> Head
        {
            get;
            private set;
        }

        /// <summary>
        /// The last node in the list or null if empty
        /// </summary>
        public DoublyLinkedListNode<T> Tail
        {
            get;
            private set;
        }

        #region Add
        /// <summary>
        /// Adds the specified value to the start of the linked list
        /// </summary>
        /// <param name="value">The value to add to the start of the list</param>
        public void AddHead(T value)
        {
            AddHead(new DoublyLinkedListNode<T>(value));
        }

        /// <summary>
        /// Adds the specified node to the start of the link list
        /// </summary>
        /// <param name="node">The node to add to the start of the list</param>
        public void AddHead(DoublyLinkedListNode<T> node)
        {
            // Save off the head node so we don't lose it
            DoublyLinkedListNode<T> temp = Head;

            // Point head to the new node
            Head = node;

            // Insert the rest of the list behind the head
            Head.Next = temp;

            if (Count == 0)
            {
                // if the list was empty then Head and Tail should
                // both point to the new node.
                Tail = Head;
            }
            else
            {
                // Before: Head -------> 5 <-> 7 -> null
                // After:  Head -> 3 <-> 5 <-> 7 -> null
                
                // temp.Previous was null, now Head
                temp.Previous = Head;
            }

            Count++;
        }

        /// <summary>
        /// Add the value to the end of the list
        /// </summary>
        /// <param name="value">The value to add</param>
        public void AddTail(T value)
        {
            AddTail(new DoublyLinkedListNode<T>(value));
        }

        /// <summary>
        /// Add the node to the end of the list
        /// </summary>
        /// <param name="value">The node to add</param>
        public void AddTail(DoublyLinkedListNode<T> node)
        {
            if (Count == 0)
            {
                Head = node;
            }
            else
            {
                Tail.Next = node;
                
                // Before: Head -> 3 <-> 5 -> null
                // After:  Head -> 3 <-> 5 <-> 7 -> null
                // 7.Previous = 5
                node.Previous = Tail;
            }

            Tail = node;
            Count++;
        }
        #endregion

        #region Remove
        /// <summary>
        /// Removes the first node from the list.
        /// </summary>
        public void RemoveHead()
        {
            if (Count != 0)
            {
                // Before: Head -> 3 <-> 5
                // After:  Head -------> 5

                // Head -> 3 -> null
                // Head ------> null
                Head = Head.Next;

                Count--;

                if (Count == 0)
                {
                    Tail = null;
                }
                else
                {
                    // 5.Previous was 3, now null
                    Head.Previous = null;
                }
            }
        }

        /// <summary>
        /// Removes the last node from the list
        /// </summary>
        public void RemoveTail()
        {
            if (Count != 0)
            {
                if (Count == 1)
                {
                    Head = null;
                    Tail = null;
                }
                else
                {
                    // Before: Head --> 3 --> 5 --> 7
                    //         Tail = 7
                    // After:  Head --> 3 --> 5 --> null
                    //         Tail = 5
                    // Null out 5's Next pointer
                    Tail.Previous.Next = null;
                    Tail = Tail.Previous;
                }

                Count--;
            }
        }
        #endregion

        #region ICollection

        /// <summary>
        /// The number of items currently in the list
        /// </summary>
        public int Count
        {
            get;
            private set;
        }

        /// <summary>
        /// Adds the specified value to the front of the list
        /// </summary>
        /// <param name="item">The value to add</param>
        public void Add(T item)
        {
            AddHead(item);
        }

        public DoublyLinkedListNode<T> Find(T item)
        {
            DoublyLinkedListNode<T> current = Head;
            while (current != null)
            {
                // Head -> 3 -> 5 -> 7
                // Value: 5
                if (current.Value.Equals(item))
                {
                    return current;
                }

                current = current.Next;
            }

            return null;
        }

        /// <summary>
        /// Returns true if the list contains the specified item,
        /// false otherwise.
        /// </summary>
        /// <param name="item">The item to search for</param>
        /// <returns>True if the item is found, false otherwise.</returns>
        public bool Contains(T item)
        {
            return Find(item) != null;
        }

        /// <summary>
        /// Copies the node values into the specified array.
        /// </summary>
        /// <param name="array">The array to copy the linked list values to</param>
        /// <param name="arrayIndex">The index in the array to start copying at</param>
        public void CopyTo(T[] array, int arrayIndex)
        {
            DoublyLinkedListNode<T> current = Head;
            while (current != null)
            {
                array[arrayIndex++] = current.Value;
                current = current.Next;
            }
        }

        /// <summary>
        /// True if the collection is readonly, false otherwise.
        /// </summary>
        public bool IsReadOnly
        {
            get
            {
                return false;
            }
        }

        /// <summary>
        /// Removes the first occurance of the item from the list (searching
        /// from Head to Tail).
        /// </summary>
        /// <param name="item">The item to remove</param>
        /// <returns>True if the item was found and removed, false otherwise</returns>
        public bool Remove(T item)
        {
            DoublyLinkedListNode<T> found = Find(item);
            if (found == null)
            {
                return false;
            }

            DoublyLinkedListNode<T> previous = found.Previous;
            DoublyLinkedListNode<T> next = found.Next;

            if (previous == null)
            {
                // we're removing the head node
                Head = next;
                if(Head != null)
                {
                    Head.Previous = null;
                }
            }
            else
            {
                previous.Next = next;
            }

            if(next == null)
            {
                // we're removing the tail
                Tail = previous;
                if(Tail != null)
                {
                    Tail.Next = null;
                }
            }
            else
            {
                next.Previous = previous;
            }

            Count--;

            return true;
        }

        /// <summary>
        /// Enumerates over the linked list values from Head to Tail
        /// </summary>
        /// <returns>A Head to Tail enumerator</returns>
        public System.Collections.Generic.IEnumerator<T> GetEnumerator()
        {
            DoublyLinkedListNode<T> current = Head;
            while (current != null)
            {
                yield return current.Value;
                current = current.Next;
            }
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return GetEnumerator();
        }

        public System.Collections.Generic.IEnumerable<T> GetReverseEnumerator()
        {
            DoublyLinkedListNode<T> current = Tail;
            while(current != null)
            {
                yield return current.Value;
                current = current.Previous;
            }
        }

        /// <summary>
        /// Removes all the nodes from the list
        /// </summary>
        public void Clear()
        {
            Head = null;
            Tail = null;
            Count = 0;
        }

        #endregion

        public bool GetHead(out T value)
        {
            if (Count > 0)
            {
                value = Head.Value;
                return true;
            }

            value = default(T);
            return false;
        }

        public bool GetTail(out T value)
        {
            if (Count > 0)
            {
                value = Tail.Value;
                return true;
            }

            value = default(T);
            return false;
        }
    }
}
```

Careful with loops.

## Sorted List

If we want a sorted list, adding in both ways makes no sense. We do not choose where elements go. We just need to go along the linked list until we find a value that is bigger or smaller.

## Deque

Double Ended queue: people can be added to the queue from both sides and can go out from both sides. You can program a Deque with a Doubly Linked List.

## Binary search tree / SortedDictionary

In C#, there is the SortedDictionary class that behaves like a binary search tree.

Binary tree with one rule: if the value of the new node is lesser than that of the current node, insert it on the left. The biggest value is the value the most on the right and the smallest is the one the most on the left.

**Complexity for insertion**: on average O(log(n)) and maximum is O(n).

### Traversal

3 types de traversée: préfixe (pre-order), infixe (in-order), postfixe (post-order). Depends on the order in which you visit the left node, current node and right node. pre-order is visit current node first.

pre-order traversal lets you make a copy of the tree. In-order lets you get the values in order. Post-order lets you delete safely a binary order tree.

### deleting a node with two children

Deleting a node with two children is like deleting the root because the children of a binary search tree form a binary search tree. Check the child on the right and its child on the left most side. This is the node we use to replace our root. He fills in all the good properties.
