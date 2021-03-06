# apply

```
Gamma contains H: \forall X1 PHI1 ->
                  \forall X2 PHI2 -> ... -> \forall Xn . P
Gamma |- PHI1[Theta1]
Gamma |- PHI1[Theta1][Theta2]
...
Gamma |- PHIn[Theta1]...[Thetan]
where P matches Q with substitution Theta1 ... Thetan
--------------------------------------------------------------------
Gamma |- Q

```

```k
module STRATEGY-APPLY
  imports PROVER-CORE
  imports STRATEGIES-EXPORTED-SYNTAX
  imports LOAD-NAMED-SYNTAX
  imports SYNTACTIC-MATCH-SYNTAX
  imports INSTANTIATE-ASSUMPTIONS-SYNTAX

  rule <k> (.K => loadNamed(Name))
               ~> apply(Name, _) ...
       </k>

    rule <k>
          (A:Pattern ~> apply(_, Strat))
          => #apply1(
               A,
               syntacticMatch(
                 terms: G, .Patterns,
                 patterns: getConclusion(A), .Patterns,
                 variables: getUniversallyQuantifiedVariables(A)
               ),
               Strat
             )
         ...</k>
         <claim> G </claim>


  syntax KItem ::= #apply1(Pattern, MatchResult, Strategy)

  rule <k>
         #apply1(A, #matchResult(subst: Subst), Strat)
         => #apply2(instantiateAssumptions(GId, Subst, A), Strat, success)
       ...</k>
       <id> GId </id>

  rule <k>
         #apply1(_, #error(_), _) => fail
       ...</k>

  syntax KItem ::= #apply2(
                     InstantiateAssumptionsResult,
                     Strategy, Strategy)

  rule <k>
         #apply2(#instantiateAssumptionsResult(.Patterns, .Map), _, Result)
         => Result
       ...</k>

  rule <k>
         #apply2(
           #instantiateAssumptionsResult(P, Ps => Ps, .Map),
           Strat,
           Result => Result & subgoal(P, Strat)
         )
       ...</k>

endmodule
```
