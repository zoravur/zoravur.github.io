+++
title = 'An Introductory Verification Exercise'
date = 2026-06-04T15:44:42-04:00
draft = false
+++

_[Github Gist](https://gist.github.com/zoravur/c80c1eb4636f1a335d764b8e97b47e6d)_

In the past month, I've been learning Lean4, because of my interest in software verification.
Here's what I did that really made it click for me:

1. Read (skim) functional programming in Lean4
2. Read (skim) theorem proving in Lean4
3. Read the first part of this textbook [Functional Algorithms, Verified!](https://www21.in.tum.de/teaching/fds/SS21/assets/book-draft.pdf) on insertion sort.
4. Straightforwardly translate the lemmas from Math to Lean4, asking ChatGPT for syntax help, and bash your 
head against Lean4's typechecker until you've verified all the lemmas. Here's the types of all the lemmas you need to prove:

```lean
import Mathlib.Data.Nat.Basic

-- variable {α : Type} [Ord a]
variable {α : Type} [LinearOrder α] [DecidableRel (· ≤ · : α → α → Prop)]

@[simp]
def insort (a : α) (as : List α) : List α :=
  match as with
  | [] => [a]
  | (y::ys) => if a ≤ y then a :: insort y ys else y :: insort a ys

@[simp]
def isort (as : List α) : List α :=
  match as with
  | [] => []
  | (x::xs) => insort x $ isort xs

def insort_is_perm
    (tail : List α) (head : α) :
    (head :: tail).Perm (insort head tail) := by
  sorry

def insertion_sort_is_perm
    (l : List α) :
    List.Perm l (isort l) := by
  sorry

theorem insort_ne_nil
    (x : α) (xs : List α) :
    insort x xs ≠ [] := by
  sorry

def isort_ne_list_is_insort_head
    (ys : List α) (y : α) :
    List.Pairwise (fun x1 x2 => x1 ≤ x2) (y :: ys) →
      y :: ys = insort y ys := by
  sorry

def mem_insort
    (a : α) (y : α) (ys : List α) :
    a ∈ insort y ys → a = y ∨ a ∈ ys := by
  sorry

def insort_is_sorted
    (head : α) (tail : List α) :
    List.Pairwise (· ≤ ·) tail →
      List.Pairwise (. ≤ .) (insort head tail) := by
  sorry

def insertion_sort_is_sorted
    (l : List α) :
    List.Pairwise (. ≤ .) (isort l) := by
  sorry

def sort_is_valid
    {α : Type}
    [LinearOrder α]
    [DecidableRel (· ≤ · : α → α → Prop)]
    (sort_fn : List α → List α) :
    Prop :=
  ∀ l : List α,
    (List.Perm l (sort_fn l)) ∧
      List.Pairwise (fun x y : α => x ≤ y) (sort_fn l)

def isort_valid :
    sort_is_valid (α := α) isort := by
  sorry
```

My solution is approximately 150 lines.
