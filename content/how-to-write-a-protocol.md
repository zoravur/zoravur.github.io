+++
title = 'How to Write a Protocol'
date = 2026-05-11T14:38:19-04:00
draft = true
tags = ["software-engineering"]
+++

In this post, we will design an application protocol from first principles. Before diving in,
it's useful to know a little bit about networking and common application level protocols. REST, in 
particular, is the foundation for how to think about application protocols. If you don't know how REST
works, go back, implement REST in a few web applications, and come back.

Ie's easy to get bogged down in complexity. For example, questions include:

- What type of transport should I use? HTTP? Sockets?
- What data format should I use? Text? Binary? Perhaps I should just throw around JSON. Or perhaps 
  protocol buffers.
- Should I model my protocols as RPC? Why not ignorable events?
- Should I use a framework? OpenAPI? gRPC? Should I use codegen?

And all of this is not even to mention the actual complexities of the logic you're trying to implement.

Whenever I get bogged down in implementation details like this, I always think, "what's the simplest thing
that could possibly work?" More often than not, that's what I end up going with, and it works fine.

So, let's start extremely simple. _What should our protocol do?_. Well, here, let's take a working example
of designing a collaborative spreadsheet editor, like Google Sheets. Our protocol should allow people to
edit a spreadsheet collaboratively. That feels a bit circular. Maybe we say, "Our protocol should facilitate
collaborative spreadsheet editing". Is that better? I'm not sure. Let's be more specific.

Our question of what the protocol should do, is really a systems design question. It's just another way
of saying, _"what are the requirements?"_

In that sense, the question can be broken down like it's an interview, into two smaller questions:

1. What are the functional requirements?
2. What are the non-functional requirements?

### Functional requirements

Functional requirements are fairly straightforward. It's the functionality that the system should support.
Obviously, people should be able to view and edit the spreadsheet. Because it is collaborative, changes 
made by one user should sync automatically between all the spreadsheets. If a user edits the spreadsheet
while offline, their edits should be incorporated seamlessly when they come back online. Also, the 
interface should support users being able to see each other on the spreadsheet. Both the user icon
indicating their presence, and their cursor specifying which cell they are currently viewing or editing.

#### What does this mean for our protocol?

Because a protocol is concerned with communication between nodes (computers), it's helpful to ask at which 
points will information need to cross a network boundary. In the above, here's what I can identify:

1. **Fetching the initial spreadsheet from the server:** Fetching the initial spreadsheet is likely the 
simplest task we'll encounter. Because a spreadsheet is a resource, like a file or dataabase row, it makes 
sense to 


| Event | Data transmitted | 
|---|---|
| Fetching the initial spreadsheet from the server | The spreadsheet object |
| Editing the spreadsheet and persisting changes to the server | The edit (if editing logic is on server), or the new spreadsheet (if not) |
| Syncing changes arriving at the server between all the clients | The edit

Here, the above exercise has already cleared up a non-trivial question about the system -- whether or not 
the default payload should be a spreadsheet or an edit. E


3.



#### What should our protocol _not_ do?





It's also helpful to define what's not in scope. 



To address the non-functional requirements
