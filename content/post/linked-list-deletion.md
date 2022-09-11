---
title: "Linked List Deletion"
date: 2022-09-11T11:14:04-04:00
draft: false
---

# What is a linked list?
A linked list is similar to an array but stores information a bit differently in memory. A linked list is a collection of nodes that contain two things:

```c
struct node {
    int data;
    struct node* next;
}
```

The ```next``` attribute is a pointer that refers to the next node in the list. This way we can traverse a linked list by using the next element of each node. All we need to store is the first node.

```c
struct node* head;
```

# Adding elements to the list.
One of the benefits of a linked list over an array is we can add elements on-the-fly without reallocating a new memory block.

All we have to do is create a new node, give it some data, and hook up the previous tail of the list's next pointer to this new node.

```c
void append(int data) {
    // create new node
    struct node* newNode;
    newNode->data = data;

    // special case where the list is empty
    if (head == NULL) {
        head = newNode;
        return;
    }

    // find tail
    struct node* tail = head;
    while (tail->next != NULL) {
        tail = tail->next;
    }

    // "link" node into the list
    tail->next = newNode;
}
```

# Deleting elements from the list.
Deleting elements is a bit more challenging. Especially if we allow deletions at any index in the list. To delete, we have to do a bit of stitching, disconnecting and reconnecting some next pointers. An example list might look like this,

> [-1] -> [3] -> [10]

To delete from this list I might create a function like this,

```c
void remove(int data) {
    // do the removal
}
```

This function will delete the first instance it finds of ```data``` in the list. For example, if we call ```remove(3);``` this list should look like,

> [-1] -> [10]

So whats required to delete a node? Take a second and think about it.

*thinking*

First we have to find the element we want to remove. If no element matches ```data``` then we leave the list as is.

```c
void remove(int data) {
    
    // the list is empty, nothing to delete
    if(head == NULL) {
        return;
    }

    // find element that matches data
    struct node* target = head;
    while (target->data != data) {
        target = target->next;

        // data does not match any element in list
        if (target == NULL) {
            return;
        }
    }
}
```

Okay, so we've written an algorithm to find an element that matches ```data```. If no match exists, we just return.

So how do we delete the ```target``` node now? Let's change the previous node's next pointer to hold target's next node.

Before
> ... -> PREV -> TARGET -> NEXT -> ...

After
> ... -> PREV -> NEXT ... 

Now when we try to traverse the list, we will never get to target because no other node has it as their next pointer. Just to be clear, the target node still exists. It is just no longer "linked" into the list. We will have to worry about freeing its memory. Let's improve our code.

```c
void remove(int data) {
    
    // the list is empty, nothing to delete
    if(head == NULL) {
        return;
    }

    // find element that matches data
    // also keep track of the previous node
    struct node* prev = NULL;
    struct node* target = head;
    while (target->data != data) {
        prev = target;
        target = target->next;

        // data does not match any element in list
        if (target == NULL) {
            return;
        }
    }

    if (target == head) {
        head = target->next;
    } else {
        prev->next = target->next;
    }

    free(target);
}
```

Hopefully you better understand deleting from linked lists now!