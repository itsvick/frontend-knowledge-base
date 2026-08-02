# Frontend Architecture

> Part of: 11-System-Design

## Overview

Frontend architecture patterns describe how UI, state, and logic are organized and kept in sync as an application grows — from classic MVC-style separation, through framework-specific variants like MVVM, to today's dominant component-based model with dedicated state-management layers.

## Key Concepts

- MVC (Model-View-Controller) separates data/business logic (Model), UI rendering (View), and user-input handling (Controller), with the Controller mediating between the two.
- MVVM (Model-View-ViewModel) — the pattern frameworks like Angular are often described as following — replaces the manual Controller with a ViewModel (the component class) that binds data to the template, and the framework's change detection keeps View and ViewModel in sync automatically.
- Flux/Redux's unidirectional data flow (Action → Reducer → Store → View) emerged as a reaction against MVC's bidirectional bindings becoming hard to trace as apps scaled.
- Component-based architecture — self-contained units of UI + logic + state composed into a tree — is the dominant modern pattern and has partly superseded classic MVC framing for frontend specifically.

## Interview Questions & Answers

### Q1. What architectural patterns have you seen used in frontend applications (e.g. MVC)?

#### Answer

MVC is the classic pattern: the Model holds data and business logic, the View renders the UI, and the Controller mediates user input, updating the Model, which in turn updates the View. MVC was historically influential on early frontend frameworks, including early Angular.

In practice, though, Angular (and similar frameworks) is more accurately described as MVVM — Model-View-ViewModel. The "ViewModel" is the component class, which binds data to the template; instead of a Controller manually pushing updates to the View, the framework's change detection automatically keeps the View and ViewModel in sync whenever the underlying data changes.

Flux/Redux introduced a different reaction to the same problem MVC has at scale: as apps grow, MVC's bidirectional data flow (View updates Model, Model updates View, Controller sits in between) becomes hard to trace, since data can change from many directions. Flux/Redux (popularized by React) enforces strictly unidirectional data flow instead: an Action is dispatched, a Reducer computes new state from it, the Store holds that state, and the View re-renders from the Store — data only ever flows one way, which makes state changes far easier to reason about and debug.

More generally, component-based architecture — self-contained units combining UI, logic, and local state, composed into a tree — is the dominant pattern in modern frontend development, and has partly superseded the classic MVC framing altogether. What used to be "Controller" responsibility is now usually split between component classes (handling view-local logic) and dedicated state-management stores/services (handling shared application state), rather than living in one distinct layer.

#### Follow-up Questions

- Why did Flux/Redux enforce unidirectional data flow instead of keeping MVC's bidirectional bindings?
- Where does "controller" logic live in a component-based architecture if there's no explicit Controller layer?

## Common Pitfalls

- Assuming all modern frameworks are "MVC" when most are more accurately MVVM or a component-based model with no distinct Controller layer at all.
- Conflating Flux/Redux's unidirectional flow with MVC's Controller-mediated flow — they solve the same coordination problem in fundamentally different ways.
- Overlooking that component-based architecture doesn't eliminate the need for a state layer — it just relocates where state and business logic live (component vs. store/service).
