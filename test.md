FormKit Core - AI Handover Document

Purpose

This document provides architectural context for building FormKit Core, an internal enterprise form platform for React-based trading applications.

This is not intended to replace React Hook Form or other ecosystem libraries. The goal is to build a configuration-driven orchestration layer capable of powering highly dynamic, high-performance enterprise forms while remaining agnostic to state management.

---

Background

Our existing internal library ("FormRender") has become difficult to maintain because it mixes together:

- Rendering
- State management
- Business logic
- Configuration

It also suffers from significant performance issues because updates to external state cause entire forms to re-render.

The objective is to design a modern replacement focused on:

- Performance
- Extensibility
- Config-driven UI
- Enterprise trading workflows

---

Target Environment

The primary use case is enterprise trading systems.

Typical workflow:

- AG Grid displays trading data.
- User selects a row.
- A side panel opens.
- The side panel contains many collapsible sections.
- Each section contains multiple form fields.
- Editing one field often affects many other fields.
- Business calculations occur continuously.
- Final submission sends the entire execution draft.

This is significantly more complex than a traditional CRUD form.

---

Primary Design Goals

- Configuration-driven UI
- Excellent rendering performance
- Fine-grained subscriptions
- Dynamic schemas
- Cross-field interactions
- Async data loading
- Dynamic select options
- Async typeahead
- External store integration
- Reusable component registry
- Enterprise scalability

---

High-Level Architecture

The architecture should be composed of independent engines.

                FormKit Core
                      │
      ┌───────────────┼────────────────┐
      │               │                │
      ▼               ▼                ▼
 UI Engine     Behavior Engine   State Engine
      │                                │
      │                   RHF Engine / Store Engine
      │
      ▼
Component Registry

                 ▲
                 │
          Data Provider

Every engine has a single responsibility.

---

Engine Responsibilities

UI Engine

Responsible for:

- Rendering from configuration
- Layout
- Groups
- Sections
- Field composition

Not responsible for:

- Business logic
- Business state
- API calls

---

State Engine

Responsible for:

- getValue()
- setValue()
- subscribe()
- validation
- dirty state
- touched state
- errors

Initial implementations:

- React Hook Form Engine
- External Store Engine

Future implementations should be possible without changing FormKit.

Examples:

- Zustand
- Redux Toolkit
- MobX
- Jotai

---

Behavior Engine

Responsible for executing declarative behaviors.

Examples:

- setValue
- clearValue
- replaceSchema
- refreshData
- validate
- focusField
- enableField
- disableField
- execute custom action

Important:

Behaviors should be declarative.

Avoid arbitrary imperative callbacks whenever possible.

Example:

Product Type changes

↓

Behavior Engine

↓

Replace schema

↓

Refresh dependent fields

↓

Update values

↓

Recalculate business data

---

Data Provider

Responsible for:

- Async select options
- Async typeahead
- API requests
- Caching
- Loading
- Debouncing
- Request cancellation
- Error handling

The schema should describe what data is needed.

The provider decides how to obtain it.

Supported backends should include:

- REST
- GraphQL
- React Query
- RTK Query
- Internal services

---

Component Registry

Responsible for mapping schema components to actual React components.

Example:

TextInput

↓

GS Text Input

Select

↓

GS Select

DatePicker

↓

GS Date Picker

Applications should be able to register custom components.

---

Field Schema Philosophy

Every field should be completely self-contained.

A field definition should include:

- rendering
- validation
- visibility
- disabled state
- layout
- behaviors
- data source
- dependencies

Example responsibilities:

Product Type

- Render Select
- Load options
- Validate
- Replace schema
- Refresh dependent fields
- Update other values

---

Dynamic Schema

Schema replacement is a first-class capability.

Example:

Product Type

Bond

↓

Bond Schema

Equity

↓

Equity Schema

This is more than simply hiding/showing fields.

Entire sections of the form may change.

---

Cross Field Behaviors

Fields should be capable of driving other fields declaratively.

Examples:

Quantity changes

↓

Recalculate Notional

↓

Update Summary

↓

Refresh Margin

---

Product changes

↓

Clear Settlement

↓

Load Brokers

↓

Replace Allocation Schema

---

These behaviors should be described in configuration.

---

Async Fields

The platform must support:

Dynamic Select

Options loaded from backend.

Example:

Country

↓

Load States

↓

State Select updates

---

Async Typeahead

Example:

Security Search

↓

Backend API

↓

Debounced results

↓

User selects security

---

Required features:

- Debounce
- Loading indicator
- Error state
- Cancellation
- Cache

---

Performance Strategy

This is one of the primary motivations.

Avoid:

watch()

↓

Entire form renders

Prefer:

Field

↓

subscribe(field)

↓

Only affected field renders

Each field should own its own subscription.

---

Source of Truth

Two supported models:

React Hook Form

Used for:

- CRUD
- Dialogs
- Wizards
- Settings

RHF owns the state.

---

External Store

Used for:

- Trading screens
- Execution editor
- Live pricing
- Cross-widget calculations

The external store owns the state.

The form is only a projection of that state.

---

Existing UI Toolkit

Applications already use an internal GS component library.

Examples:

- Text Input
- Number Input
- Select
- Date Picker
- Async Typeahead

FormKit must integrate with these components.

It should not require adopting another UI library.

---

Existing Form Config

Applications already use config-driven forms.

The new platform should continue supporting configuration-driven rendering.

Migration from existing configs should be considered.

---

Evaluation of Existing Libraries

We investigated Formily.

Strengths:

- Excellent performance
- Dynamic schemas
- Reactive behaviors
- Async support
- Component registration

Concerns:

- Owns its own form state.
- Uses its own form engine.
- External store as the primary source of truth may require adaptation.
- Existing internal configs would likely need translation.

Current recommendation:

Do not immediately build a new framework.

Instead:

1. Build a small proof of concept using Formily.
2. Attempt to implement one representative trading screen.
3. Validate whether external store ownership integrates naturally.
4. Only build custom infrastructure if Formily fundamentally conflicts with the required architecture.

---

Non Goals

FormKit should NOT become:

- A validation library
- A networking library
- A state management library
- A replacement for React Hook Form
- A replacement for React Query

Instead, it should orchestrate those libraries.

---

Design Principles

- Configuration First
- Performance First
- State Agnostic
- Declarative Behaviors
- Composition over Inheritance
- Single Responsibility
- Single Source of Truth
- Pluggable Engines

---

Open Questions

These should be answered before implementation:

1. Can Formily use an external store as the canonical source of truth without awkward synchronization?

2. Should FormKit internally use Formily as an engine, or should it build its own UI engine?

3. How should schema migrations from the existing FormRender config be handled?

4. What plugin API should be exposed for future extensions?

---

Recommended Next Steps

1. Create a proof of concept using Formily.
2. Integrate GS internal UI components.
3. Integrate an external execution draft store.
4. Measure rendering performance.
5. Evaluate developer experience.
6. Document architectural gaps.
7. Decide whether:
   - Formily + thin wrapper is sufficient, or
   - A custom FormKit implementation is justified.

The goal is to avoid rebuilding mature infrastructure unless there is clear evidence that existing solutions cannot satisfy the enterprise requirements.