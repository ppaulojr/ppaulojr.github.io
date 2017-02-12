---
layout: post
title:  "Questions for interviewing devs"
date:   2016-02-11 15:00:00 -0300
categories: interview
---

One tricky thing is to come out with a plan to interview developers. Back in 1999 I got a job offer from Microsoft based in a full day in person interview which was in my opinion one of the best crafted hiring process I ever witnessed, however if you are not Microsoft, Google or Apple you cannot afford motivating people to take full day interviews or flying 10 engineers to conduct the process.

The post is not about what you should do or avoid but to comment on some questions that I came across during those years in the job.

> How do you reverse a linked list?

The solution is easy for any computer science student but can be tricky when asked when a candidate would be expecting a higher lever interview and is caught by surprise, like the move 1. e4 e5 2.Qh5 in a bullet chess game.

It's an okay question for an interview, but before asking I would take the following precautions:

1. Set the candidate at easy saying we don't expect a textbook answer but just to see how they think.
2. Give some help if the interviewed person is stuck because we caught them off guard
3. Sometimes it's helpful to give some context like:

```c
struct node
{
    int data;
    struct node* next;
};
```

What is not okay in this case is that we expect the candidate to came up with a textbook canonical answer like:

```c
static void reverse(struct node** head_ref)
{
    struct node* prev   = NULL;
    struct node* current = *head_ref;
    struct node* next;
    while (current != NULL)
    {
        next  = current->next;  
        current->next = prev;   
        prev = current;
        current = next;
    }
    *head_ref = prev;
}
```

We could see some people came up with elegant, although not the best solution:

```python
A = Stack()
for i in list_:
   A.push(i)

l = List()
while !A.isEmpyt():
   e = A.pop()
   l.append(e)
```

And then we have a problem: which candidate is better? The one with a textbook solution or the other? 

I believe the question of which candidate is better based on this question is pointless. The real question is what kind of intel I gained by asking it? Am I making my hiring process more robust by asking it or not.

While I concede it's an okay question I think it introduces more noise than signal in your process and I hope in future posts to craft other questions that I used to sort out got fits for the positions I was hiring people for.