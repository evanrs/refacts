# Refacts
A state container built on facts

## Motivation
Current solutions for application state are too imperative.

As an example consider a Todo app using Redux:
- user submits todo
- an action is dispatched to add a todo
- each reducer compares the action type (a string constant) to its own string constants
- once the todo list reducer is reached (mathcing its type) it puts the todo at the head of a list concating the previous list
- the store observes a change, updating its subscribers
- a subscriber will receive the update by selecting a hard coded fragment of the state
- if that fragment has changed those changes will be reflected
- if it hasn't changed that will also be reflected, this depends on how naive the subsriber is

This explodes in complexity for an sufficiently advanced app. It also largely has little to do with the intent of the user.

An idealized example:
- user submits todo
- a fact is created, there exists a todo(x)
- known subsets matching this fact are notified
- a subscriber of a known subset reflects the changes

We reify the fact as a member of subsets, these subsets provide identiy criteria for membership.

A sandpaper grade rough example:
```es6
/**
 * Kind
 * Instantiation has the intrinsic properties of `created`, `updated`, `deleted`,
 *
 * These properties are reified as attributes:
 *   createdAt, updatedAt, and deletedAt
 *
 * The intuitive Quality of these properties is temporal, so they can be used
 * in a formal relation such as ordering by recently updated
 *
 * The semantics for a change to an instance are Create, Update, Delete, as in:
 *   todo = Create(Todo, { text: 'new text'})
 *   todo = Update(todo, { text: 'updated text'})
 *   todo = Delete(todo)
 */

set(
  Kind(
    'Todo', Attributes({ text: String })),
  MaterialRelation(
    'Completed', Predicate('complete')),
  FormalRelation(
    'Newer', Predicate('created')),
  Subset(
    'CompletedTodos', Kind('Todo'), Predicate('complete')),
  Subset(
    'RecentTodos', Kind('Todo'), Order('Newer')),
  Subset(
    'RecentCompletedTodos', Intersect('RecentTodos', 'CompletedTodos'))
)

set.getState() === {
  RecentTodos: [],
  CompletedTodos: [],
  RecentCompletedTodos: []
}

set.add(
  Create(Todo, { text: 'This is a todo' }))

set.getState() === {
  RecentTodos: [
    { text: 'This is a todo',
      isComplete: false,
      isNotComplete: true,
      createdAt: '...',
      deletedAt: '...'
    }
  ],
  CompletedTodos: [],
  RecentCompletedTodos: []
}
```

This project is inspired by the work on [DOLCE](https://en.wikipedia.org/wiki/Descriptive_Ontology_for_Linguistic_and_Cognitive_Engineering) and [OntoClean](https://en.wikipedia.org/wiki/OntoClean) by Nicola Guarino et al. 
