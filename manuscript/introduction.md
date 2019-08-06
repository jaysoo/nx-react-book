# Introduction

If you've ever worked at a company with more than one team, chances are you've had to deal with some challenges when it comes to change management.

In a typical work setting, development teams are divided by features or technology. For example, you may have one frontend team building the UI in React, and one backend team building the API in Express. These teams usually have their own code repositories, so changes to the software as a whole requires juggling multiple repositories.

A few problems that arise from a multi-repository setup include:

- Lack of collaboration because sharing code is hard and expensive.
- Coordinating changes across multiple module is hard.
- Bugs are discovered at the point of integration rather than when code is changed.
- Lack of consistency in linting, testing, and release processes.

## Monorepos to the rescue!

A lot of successful organizations such as Google, Facebook, and Microsoft -- and also large open source projects such as Babel, Jest, and React -- are all using the monorepo approach to software development.

As you will see in this book, a monorepo approach when done correctly can save developers from a great deal of headache and wasted time.

## Why Nx?

Nx is a set of dev tools designed specifically to help teams work with monorepos. There is an opionated organizational structure, and an opionated set of generation, linting, and testing tools.

## About this book

This book is split into three parts.

In **chapter 1** we begin by setting up the monorepo workspace with Nx and create our first application: An online bookstore. We will explore a few Nx commands that work right out of the box.

In **chapter 2** we build new libraries to support a book listing feature.

In **chapter 3** we examine how Nx deals with code changes by arming us with intelligent tools to help us understand and verify changes. We will demonstrate these Nx tools by creating an `api` backend application.

In **chapter 4** we wrap up our application by implementing the `shopping-cart` feature, where users can add book to their cart for checkout.

In **chapter 5** we will look at how we can create production bundles and deployment strategies.
