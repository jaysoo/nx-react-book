# Introduction

If you've ever worked at a company with more than one team, chances are you've had to deal with some challenges when it comes to change management.

In a typical work setting, development teams are divided by domain or technology. For example, you may have one frontend team building the UI in React, and one backend team building the API in Express. These teams usually have their own code repositories, so changes to the software as a whole requires juggling multiple repositories.

A few problems that arise from a multi-repository setup include:

- Lack of collaboration because sharing code is hard and expensive.
- Lack of consistency in linting, testing, and release processes.
- Difficulty in coordinating changes across repositories.
- Late discovery of bugs because they can only occur at the point of integration rather than when code is changed.

## Monorepos to the rescue!

A lot of successful organizations such as Google, Facebook, and Microsoft -- and also large open source projects such as Babel, Jest, and React -- are all using the monorepo approach to software development.

As you will see in this book, a monorepo approach when done correctly can save developers from a great deal of headache and wasted time.

## Why Nx?

Nx is a set of dev tools designed specifically to help teams work with monorepos. It provides an opinionated organizational structure, and an opinionated set of generation, linting, and testing tools.

## Is this book for you?

Because Nx creates TypeScript source code, it helps to know some of the basics such as type annotations, and interfaces.

```typescript
// Here's a number variable
let x: number;

// Here's an interface
interface Foo {
  bar: string;
}

// Using the interface
const y: Foo = {
  bar: 'Hello'
};
```

Don't fret if this is your first introduction to TypeScript. We will not be using any advanced TypeScript features so a good working knowledge of modern JavaScript is more than enough.

This book assumes that you have prior experience working with React, so it does not go over any of the basics. We will also make light use of the Hooks API, however understanding it is not necessary to grasp the concepts in this book.

Consequently, this book might be for you if:

- You just heard about Nx and want to know more about how it applies to React development.
- You use React at work and want to learn tools and concepts to help your team work more effectively.
- You want to use great tools that enable you to focus on product development rather than environment setup.
- You use a monorepo but have struggled with its setup. Or perhaps you want to use a monorepo but are unsure how to set it up.
- You are pragmatic person who learns best by following practical examples of an application development.

On the other hand, this book might not be for you if:

- You are already proficient at using Nx with React and this book may not teach you anything new.
- You hate monorepos so much that you cannot stand looking at them.

## How this book is laid out

This book is split into three parts.

In **chapter 1** we begin by setting up the monorepo workspace with Nx and create our first application: An online bookstore. We will explore a few Nx commands that work right out of the box.

In **chapter 2** we build new libraries to support a book listing feature.

In **chapter 3** we examine how Nx deals with code changes in the monorepo by arming us with intelligent tools to help us understand and verify changes. We will demonstrate these Nx tools by creating an `api` backend application.

In **chapter 4** we wrap up our application by implementing the `shopping-cart` feature, where users can add book to their cart for checkout.

In **chapter 5** we will look at how we can create production bundles and deployment strategies.
