+++
title = 'The Terraform Knows What It Should Provision At All Times.'
date = 2025-07-01T14:50:43-04:00
draft = false
tags = ['memes']
+++

 the terraform knows what it should provision at all times.
 it knows this because it knows what it hasn’t provisioned.
 
 by subtracting what is from what ought to be (or what ought to be from what is, depending on provider drift), it obtains a *plan*, or *diff*.
 
 the planning subsystem uses diffs to generate imperative operations to drive the infrastructure from a state that is, to a state that isn’t, and in arriving at a state that wasn’t, it now is.
 
 consequently, the state that now *is*, is not the state that *was*, and therefore the state that *was* is no longer what *is*.
 this is called *idempotence*.
 
 in the event that the state that is does not match the state that should be, the system has acquired *drift*.
 drift is the discrepancy between the actual state and the desired state, and if drift is nontrivial, the terraform will attempt *reconciliation*.
 
 however, terraform must also know what it last applied.
 
 the terraform plan execution proceeds as follows:
 because drift may have invalidated some resource attributes, terraform is not exactly sure what *is*,
 but it is sure what *should be*,
 and within tolerances, what *was*.
 
 it now subtracts what *should be* from what *is not yet*, or vice versa, and by interpolating this against the algebraic projection of what *must not be*, it obtains the *execution plan*.
 
 this plan, when applied, corrects the infra such that what *should be* *becomes*, what *was* is *forgotten*, and what *is not yet* is now *managed*.

 Happy Canada Day.
