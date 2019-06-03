What if a CCS-check has a bug? 
THis could happen if I did not test audit CCSs for a while. Can I disable it w/o assumptions?

Josh's lifecycle: ignore -> check and continue -> check and kill -> assume (all for default)

RYan's example

Josh will never turn on assumes untill he heas not turned runtime checks before.


or invocation of the violation handler and any side effects that function might have.


Natan's use case for axioms as optimizations: test whole program with axiom assumtions -- enough safety.


While it does not seem to be a big issue, I have in my notes this one: " If a function has multiple preconditions, their evaluation (if any) will be performed in the order they appear lexically."
While this is enough to determin


variant in a no-exceptions world: in the end you do not get the variant into the state you wanted , so you have to cancel the operation that needed it anyway.


--------------

[dcl.attr.contract.check]/p6

There are programs that need to resume computation after executing a violations handler. This seems counterintuitive (“continue after a precondition violation!!!?”), but there are two important use cases:

* Gradual introduction of contracts:  Experience shows that once you start introducing contracts into a large old code base, violations are found in “correctly working code.” In other words, contracts are violated in ways that did not cause crashes for the actual use of the code for the actual data used. There are examples where the number of such “currently harmless violations” is massive. The way to cope is to install a violation handler that logs the problem and continues. This allows gradual adoption.

* Test harnesses: Contracts are code, so they can contain bugs. To test contracts, we need to execute examples of violations. A convenient way of organizing such a test suite is to have a violation handler throw an exception and have the main testing loop catch exceptions and proceed running the next test


------------------------------


[assert] -> [assume] + [verify] -- maybe this does not apply to preconditions and postconditions?

"does axiom mean assume? or just documentation?"

Jason:
assert and assume they are the input and output of one feature.

Bjarne:
 "something we do not require proof for (usually because we cannot prove it), but may rely on."
 
 David Stone:
 
     I think Richard's point is an important one, and one that I think is easy to under-appreciate. That being said, I understand and I have a lot of sympathy for the ability to continue from a failed contract check. Maybe there is a way to address Richard's concern without removing the feature?

    I have worked in code bases that had lots of "defensive programming" (and I imagine most of us have encountered code like this at some point).

      bool f(int * ptr) {
        if (!ptr) {
          return false;
        }
        // do stuff using *ptr, returning true if we eventually succeed
      }

    No one is ever supposed to call f with a nullptr argument, but we put in a check to avoid crashing in case some did. This feels like the exact use-case contracts is supposed to solve, and mirrors what Ville has brought up earlier in this thread. The question is: How do I roll out such a change? So far, none of the solutions seem satisfying, which gets to Richard's point. This code has existed as-is for (potentially) decades, so it may have callers passing it nullptr and relying on the defensive check. I would like to be able to use the contracts facilities to handle this, but that gets difficult if I am just part of a larger code base.

    The current continuation yes-no semantics work when I am the first person in my code base to add contracts: I first leave continuation on and install a logging handler; then, once I am satisfied, I change it into a terminating handler. But what do I do if I am not the first person in my code base to start using contracts? I don't want to turn everyone else's contracts (which they have already tested and are confident that they can terminate on violation, and they want the compiler to optimize assuming they are true) into log-and-continue. Does this mean I cannot use the feature?

    We need something more fine-grained than "continuation mode for your whole program: yes or no?". The use cases that make sense to me are more of "This contract: continue or not?", with the long-term recommendation being "Do not continue". Some sort of functionality like this seems like the only way to support an incremental transition to contracts.

    This style of contract specification also has the advantage of localizing the logic of how optimizations work: a contract statement that does not support continuation will never attempt to execute a later contract, and thus the later contract will never optimize code before that point in the case where the earlier contract is false. A contract statement that does support continuation behaves like any other code that supports continuation: if a later statement assumes something is true, it must be true or else undefined behavior has occurred. This seems like it still fits in the use case I described, because it involves taking code that works fine now (so however it is being optimized is current behavior) and stating it again. Admittedly, minor restructurings can change how optimizers behave in the presence of undefined behavior, so it's not entirely air-tight, but this seems to move us more in a direction that, it seems, everyone wants.
    
    
Herd:
Let me try two followups:

    Can we agree that an "assert" condition documents an expectation on the program's state, where a violation means "unexpected state" -- we are in a state the program code was not designed to handle because it was never expected?

    Can we agree that if that condition is additionally "assume"d by the compiler, that elevates "unexpected state" to "language UB"?

Nathan:
    The message from Gaby, if it's the one I am thinking of,
    did not present the semantics prescribed in the WD, or the
    proposal. If you want a concise, concrete illustration:

       [[assert: a]]
       [[assert axiom: b]]

    becomes, by the standard wording,

       a || $handler();
       b || *(int*)0;

    in default-checking mode.  (You may substitute your own
    favorite UB into the second line. Is there a more
    concise expression of UB?)
    
    
[[assert: p != nullptr]]
[[assert axiom: p != nullptr]]


MY APPOLOGIES FOR REPEATING THIS -- I PREVIOUSLY RESPONDED IN THE WRONG THREAD.

Richard Smith correctly pointed out that the Contract Assertion facility as proposed is somewhat limited in its effectiveness in that the granularity for continuation is ALL or NOTHING, which makes the general usefulness of the continuation feature questionable -- especially for larger programs.

David Stone has reiterated his concern, correctly pointing out exactly why it is a problem for larger systems, such as those found at companies like Bloomberg. It should come as no surprise that we are aware of these limitations, and are way out in front of it with follow-up work that needs to be proposed to complete what we have now and make runtime contract validation in C++ all that it was meant to be.

To that end, I asked Joshua Berne, lead programmer on this effort from Bloomberg to write something up to explain what we are doing here at Bloomberg, and where we see contracts going in the future.  I am posting it here unedited on Josh's behalf.

                                   - - - - - 
Josh Berne (Bloomberg LP) Wrote:
This problem of introducing assertions into already deployed code is one that we've been addressing here at Bloomberg with a new facility ('BSLS_REVIEW').   This is a form of assertion that by default just logs and returns instead of by default aborting.   Reviews have a distinct failure handler from assertions, and, for the workflows we've encountered, having 2 parallel facilities fully addresses the lifecycles of an assertion through the production release cycle. 

When introducing a new check into existing code, it is first put in as a BSLS_REVIEW, released and monitored.  Once we have verified that the contract in question is well formed and not being violated we can then just transition the BSLS_REVIEW macro into a BSLS_ASSERT macro (carefully named so that we don't even have to reformat the code - sadly we're not in a fully clang-format driven world here at Bloomberg yet.)

The advantage of a separate handler for review is that we incorporate setting that into our testing strategy - In tests, the default for review failures is also to abort.  This lets us catch failures fast in those kinds of deployments immediately instead of waiting until the code happens to run in production with the review inserted.  The other advantage of it being distinct is what addresses David Stone's concern - the review being added and having a distinct default behavior does not impact how other existing assertion failures in the code get handled.  

Basically, at the code level we have 2 distinct options that capture our intention for a check - either it is a temporary check that we want evaluated/logged but that we don't want to impact control flow, or it is a 'confirmed' check that we expect to always be false and want to abort on.  This is not something currently in the existing contracts proposal, so the workflow of adding new checks to existing code is not a smooth one (largely due to the single monolithic switch on assertion behavior.)  We are planning to do that after getting more experience with how 'BSLS_REVIEW' works and seeing if there are any further needs that come out of the weeds now that we have it deployed at scale.

- - - - - 

Fabio Fracassi:

    One of the most stable Systems I have worked on has the same facility under the name ASSERT_NOTIFY. It was used not only as a transition measure towards "full" ASSERTS (and thus I would prefer [[assert notify: ...]] or [[assert log: ...]] but that bikeshed can be painted another day), but it proved very useful whenever a mistaken assumption does or might lead to bugs, but not necessarily to fatal ones. 

    As a consequence it also leads programmers to be more confident in documenting their assumptions through assertions.


    
Are contract supposed to be used for dead code elimination by the compiler?

