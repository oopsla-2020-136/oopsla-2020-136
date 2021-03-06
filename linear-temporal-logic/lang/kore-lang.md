Kore Sugar
==========

```k
module TOKENS
  // Lexical
  syntax UpperName
  syntax LowerName ::= CheckSATResult
  syntax SharpName
  syntax ColonName
  syntax PipeQID
  syntax Decimal

  // Abstract
  syntax Symbol ::= UpperName | LowerName
  syntax VariableName ::= UpperName
  syntax Sort ::= UpperName | LowerName

  syntax CheckSATResult ::= "sat"       [token] 
                          | "unsat"     [token]
                          | "unknown"   [token]

  // Identifiers used directly in the semantics

  syntax LowerName ::= "emptysetx"  [token]
                     | "unionx"     [token]
                     | "singleton"  [token]
                     | "intersectx" [token]
                     | "in"         [token]
                     | "disjointx"  [token]
                     | "const"      [token]
                     | "n"          [token]
                     | "x"          [token]
                     | "s"          [token]
                     | "y"          [token]
                     | "ite"        [token]
                     | "setAdd"     [token]
                     | "setDel"     [token]
                     | "setminus"   [token]
                     | "max"        [token]

  syntax LowerName ::= "emptyset"      [token]
                     | "singleton"     [token]
                     | "union"         [token]
                     | "disjoint"      [token]
                     | "disjointUnion" [token]
                     | "isMember"      [token]
                     | "add"           [token]
                     | "del"           [token]

  // sep-logic symbols
  syntax LowerName ::= "pto" [token]
                     | "sep" [token]
                     | "nil" [token]
                     | "emp" [token]
  syntax Symbol ::= "nil" "{" Sort "}"

  // Arith
  syntax LowerName ::= "plus"          [token]
                     | "minus"         [token]
                     | "mult"          [token]
                     | "div"           [token]
                     | "lt"            [token]
                     | "gt"            [token]
                     | "lte"           [token]
                     | "gte"           [token]
                     | "max"           [token]

  // TODO: These aren't LowerNames
  syntax LowerName ::= "*"     [token]
                     | "+"     [token]
                     | "/"     [token]
                     | "-"     [token]
                     | "^"     [token]
                     | ">"     [token]
                     | "<"     [token]
                     | ">="    [token]
                     | "<="    [token]

  // Array
  syntax LowerName ::= "store"         [token]
                     | "select"        [token]

  // Core symbols
  syntax LowerName ::= "not"      [token]
                     | "or"       [token]
                     | "and"      [token]
                     | "=>"       [token]
                     | "="        [token]
                     | "=>"       [token]
                     | "ite"      [token]
                     | "distinct" [token]

  // Sets (defined by CVC4, but not Z3)
  syntax LowerName ::= "emptyset"     [token]
                     | "singleton"    [token]
                     | "union"        [token]
                     | "intersection" [token]
                     | "member"       [token]

  // Extensional Arrays
  syntax LowerName ::= "select" [token]
                     | "store"  [token]
                     | "map"    [token]

  // Sorts
  syntax UpperName ::= "Array" [token]
                     | "ArrayIntInt" [token]
                     | "Bool" [token]
                     | "Heap" [token]
                     | "Int" [token]
                     | "Set" [token]
                     | "SetInt" [token]
endmodule

module TOKENS-SYNTAX
  imports TOKENS
  syntax UpperName ::= r"[A-Z][A-Za-z\\-0-9'\\#\\_]*"  [prec(100), token, autoReject]
  syntax LowerName ::= r"[a-z][A-Za-z\\-0-9'\\#\\_]*"  [prec(100), token, autoReject]
  syntax SharpName ::= r"#[A-Za-z\\-0-9'\\#\\_]*"  [prec(100), token, autoReject]

  syntax ColonName ::= r":[a-z][A-Za-z\\-0-9'\\#\\_]*" [token, autoReject]
  syntax Decimal ::= r"[0-9][0-9]*\\.[0-9][0-9]*" [token, autoreject]
                   | "2.0" [token]
endmodule

module KORE
  imports TOKENS
  imports INT-SYNTAX
  imports BOOL-SYNTAX
  imports STRING-SYNTAX

  syntax Ints ::= List{Int, ","}
```

We allow two "varieties" of variables: the first, identified by a String, is for
use in defining claims; the second, identified by a String and an Int subscript
is to be used for generating fresh variables. *The second variety must be used
only in this scenario*.

```k
  syntax Variable ::= VariableName "{" Sort "}" [klabel(sortedVariable)]
  syntax SetVariable ::= SharpName [klabel(setVariable)]
  syntax Context ::= VariableName "[" Pattern "]" [klabel(context)]
  syntax Pattern ::= Int
                   | Variable
                   | SetVariable
                   | Symbol
                   | Symbol "(" Patterns ")"                    [klabel(apply)]

                   | Context

                   | "\\top"    "(" ")"                         [klabel(top)]
                   | "\\bottom" "(" ")"                         [klabel(bottom)]
                   | "\\equals" "(" Pattern "," Pattern ")"     [klabel(equals)]
                   | "\\not"    "(" Pattern ")"                 [klabel(not)]

                   | "\\and"    "(" Patterns ")"                [klabel(and)]
                   | "\\or"     "(" Patterns ")"                [klabel(or)]
                   | "\\implies" "(" Pattern "," Pattern ")"    [klabel(implies)]

                   | "\\exists" "{" Patterns "}" Pattern        [klabel(exists)]
                   | "\\forall" "{" Patterns "}" Pattern        [klabel(forall)]

                   | "\\mu" SetVariable "." Pattern             [klabel(mu)]
                   | "\\nu" SetVariable "." Pattern             [klabel(nu)]
                     /* Sugar for \iff, \mu and application */
                   | "\\iff-lfp" "(" Pattern "," Pattern ")"    [klabel(ifflfp)]

                   | "\\member" "(" Pattern "," Pattern ")"     [klabel(member)]
                   | "\\subseteq" "(" Pattern "," Pattern ")"   [klabel(subseteq)]

                   // sugar for commonly needed axioms
                   | "\\typeof" "(" Pattern "," Sort ")"
                   | "functional" "(" Symbol ")"
                   | "constructor" "(" Symbol ")"
                   | "partial" "(" Patterns ")"
                   | "heap" "(" Sort "," Sort ")" // Location, Data
                   | "\\hole" "(" ")" [klabel(Phole)]
                   // \functionalPattern(Phi) means \exists x. Phi=x
                   | "\\functionalPattern" "(" Pattern ")"

  rule \top()    => \and(.Patterns) [anywhere]
  rule \bottom() => \or(.Patterns) [anywhere]

  syntax Patterns ::= List{Pattern, ","}                        [klabel(Patterns)]
  syntax Sorts ::= List{Sort, ","}                              [klabel(Sorts)]

  syntax SymbolDeclaration ::= "symbol" Symbol "(" Sorts ")" ":" Sort
  syntax SortDeclaration ::= "sort" Sort
  syntax AxiomName ::= LowerName | UpperName
  syntax ClaimName ::= LowerName | UpperName
  syntax AxiomOrClaimName ::= AxiomName | ClaimName

  syntax Declaration ::= "imports" String
                       | "axiom" Pattern
                       | "axiom" AxiomName ":" Pattern
                       | SymbolDeclaration
                       | SortDeclaration
  syntax Declarations ::= Declaration Declarations
  syntax Declarations ::= ".Declarations" [klabel(.Declarations), symbol]

  syntax VariableName ::= "#hole"

  // Sugar for `\exists #hole . #hole /\ floor(Arg1 -> Arg2)
  syntax Pattern ::= implicationContext(Pattern, Pattern) [klabel(implicationContext)]

endmodule
```

Kore Helpers
============

Here we define helpers for manipulating Kore formulae.

```k
module KORE-HELPERS
  imports KORE
  imports MAP
  imports INT
  imports STRING
  imports PROVER-CONFIGURATION
  imports PROVER-CORE-SYNTAX
  imports VISITOR-SYNTAX

  syntax String ::= SortToString(Sort)     [function, functional, hook(STRING.token2string)]
  syntax String ::= SymbolToString(Symbol) [function, functional, hook(STRING.token2string)]
  rule SymbolToString(nil { SORT }) => SymbolToString(nil) +String "_" +String SortToString(SORT)
  syntax LowerName ::= StringToSymbol(String) [function, functional, hook(STRING.string2token)]

  syntax Bool ::= Pattern "in" Patterns [function]
  rule P in (P,  P1s) => true
  rule P in (P1, P1s) => P in P1s requires P =/=K P1
  rule P in .Patterns => false

  syntax Patterns ::= Patterns "++Patterns" Patterns [function, right]
  rule (P1, P1s) ++Patterns P2s => P1, (P1s ++Patterns P2s)
  rule .Patterns ++Patterns P2s => P2s

  syntax Declarations
  ::= Declarations "++Declarations" Declarations [function, right]
  rule (D1 D1s) ++Declarations D2s => D1 (D1s ++Declarations D2s)
  rule .Declarations ++Declarations D2s => D2s

  syntax Patterns ::= Patterns "intersect" Patterns [function]
  rule .Patterns intersect Ps => .Patterns
  rule (P, Ps1) intersect Ps2 => P, (Ps1 intersect Ps2)
    requires P in Ps2
  rule (P, Ps1) intersect Ps2 => Ps1 intersect Ps2
    requires notBool (P in Ps2)

  syntax Patterns ::= removeDuplicates(Patterns) [function]
  rule removeDuplicates(.Patterns) => .Patterns
  rule removeDuplicates(P, Ps) => P, removeDuplicates(Ps)
  requires notBool(P in Ps)
  rule removeDuplicates(P, Ps) => removeDuplicates(Ps)
    requires P in Ps

  syntax Patterns ::= Patterns "-Patterns" Patterns [function]
  rule (P1, P1s) -Patterns P2s => P1, (P1s -Patterns P2s)
    requires notBool(P1 in P2s)
  rule (P1, P1s) -Patterns P2s =>      (P1s -Patterns P2s)
    requires P1 in P2s
  rule .Patterns -Patterns P2s => .Patterns
  rule P1s -Patterns .Patterns => P1s

  syntax Patterns ::= removeFirst(Pattern, Patterns) [function]
  rule removeFirst(P, (P, Ps)) => Ps
  rule removeFirst(P1, (P2, Ps)) => P2, removeFirst(P1, Ps)
    requires P1 =/=K P2
  rule removeFirst(_, .Patterns) => .Patterns

  syntax Set ::= PatternsToSet(Patterns) [function]
  rule PatternsToSet(.Patterns) => .Set
  rule PatternsToSet(P, Ps) => SetItem(P) PatternsToSet(Ps)

```

```k
  syntax Bool ::= isFunctional(GoalId, Pattern) [function]

  rule isFunctional(_, plus #Or minus #Or select #Or union
         #Or singleton #Or emptyset) => true

  rule [[ isFunctional(_, S) => true ]]
       <declaration> axiom _: functional(S) </declaration>

  rule [[ isFunctional(GId, S) => true ]]
       <id> GId </id>
       <local-decl> axiom _: functional(S) </local-decl>

  rule isFunctional(_, _:Int) => true
  rule isFunctional(_, _{_}) => true
  rule isFunctional(GId, R(ARGS))
       => isFunctional(GId, R) andBool areFunctional(GId, ARGS)
  rule isFunctional(_,_) => false [owise]

  syntax Bool ::= areFunctional(GoalId, Patterns) [function]

  rule areFunctional(_, .Patterns) => true
  rule areFunctional(GId, P, Ps)
       => isFunctional(GId, P) andBool areFunctional(GId, Ps)

  syntax Bool ::= isConstructor(Pattern) [function]
  rule [[ isConstructor(S) => true ]]
       <declaration> axiom _: constructor(S) </declaration>

  syntax Sort ::= getReturnSort(Pattern) [function]
  rule getReturnSort( I:Int ) => Int
  rule getReturnSort( _ { S } ) => S
  rule getReturnSort(\exists{_} P) => getReturnSort(P)
  rule getReturnSort(\and((P, Ps))) => getReturnSort(P)
       requires sameSortOrPredicate(getReturnSort(P), Ps)
  rule getReturnSort( plus ( ARGS ) ) => Int
  rule getReturnSort( minus ( ARGS ) ) => Int
  rule getReturnSort( select ( ARGS ) ) => Int
  rule getReturnSort( union ( ARGS ) ) => SetInt
  rule getReturnSort( singleton ( ARGS ) ) => SetInt
  rule getReturnSort( emptyset ) => SetInt
  rule getReturnSort( disjoint ( ARGS ) ) => Bool
  rule getReturnSort( lt  ( ARGS ) ) => Bool
  rule getReturnSort( gt  ( ARGS ) ) => Bool
  rule getReturnSort( lte ( ARGS ) ) => Bool
  rule getReturnSort( gte ( ARGS ) ) => Bool
  rule getReturnSort( isMember ( _ ) ) => Bool
  rule getReturnSort( nil { SORT } ( _ ) ) => SORT
  rule [[ getReturnSort( R ( ARGS ) )  => S ]]
       <declaration> symbol R ( _ ) : S </declaration>
  rule [[ getReturnSort( R ( ARGS ) )  => S ]]
       <local-decl> symbol R ( _ ) : S </local-decl>

  syntax Bool ::= sameSortOrPredicate(Sort, Patterns) [function]

  rule sameSortOrPredicate(_, .Patterns) => true

  rule sameSortOrPredicate(S, (P, Ps))
    => sameSortOrPredicate(S, Ps)
       requires isPredicatePattern(P)

  // We basically implement short-circuiting Or to prevent
  // calling getReturnSort on predicate patterns.

  rule sameSortOrPredicate(S, (P, Ps))
    => #sameSortOrPredicate(S, (P, Ps))
       requires notBool isPredicatePattern(P)

  syntax Bool ::= #sameSortOrPredicate(Sort, Patterns) [function]

  rule #sameSortOrPredicate(S, (P, Ps))
    => sameSortOrPredicate(S, Ps)
       requires getReturnSort(P) ==K S

  rule #sameSortOrPredicate(S, (P, Ps))
    => false
       requires getReturnSort(P) =/=K S

  rule getReturnSort(\mu _ . Phi) => getReturnSort(Phi)
  rule getReturnSort(\nu _ . Phi) => getReturnSort(Phi)
  rule getReturnSort(\or(P, Ps)) => unionSort(getReturnSort(P), getReturnSort(\or(Ps)))
  rule getReturnSort(\or(.Patterns)) => BottomSort
  rule getReturnSort(\and(P, Ps)) => intersectSort(getReturnSort(P), getReturnSort(\and(Ps)))
  rule getReturnSort(\and(.Patterns)) => TopSort
  rule getReturnSort(_:SetVariable) => TopSort

  rule getReturnSort(\exists{Vs} P) => getReturnSort(P)

  syntax UpperName ::= "TopSort"         [token]
                     | "BottomSort"      [token]

  syntax Sort ::= unionSort(Sort, Sort) [function]
  rule unionSort(TopSort, S) => TopSort
  rule unionSort(S, TopSort) => TopSort
  rule unionSort(BottomSort, S) => S
  rule unionSort(S, BottomSort) => S

  syntax Sort ::= intersectSort(Sort, Sort) [function]
  rule intersectSort(TopSort, S) => S
  rule intersectSort(S, TopSort) => S
  rule intersectSort(BottomSort, S) => BottomSort
  rule intersectSort(S, BottomSort) => BottomSort

  syntax Bool ::= isUnfoldable(Symbol) [function]
  rule [[ isUnfoldable(S:Symbol) => true ]]
       <declaration> axiom _ : \forall {_} \iff-lfp(S(_), _) </declaration>
  rule isUnfoldable(S:Symbol) => false [owise]

  syntax Patterns ::= getGroundTerms(Pattern) [function]
  rule getGroundTerms(P) => getGroundTerms(P, .Patterns)
  syntax Patterns ::= getGroundTerms(Pattern, Patterns) [function, klabel(getGroundTermsAux)]
  rule getGroundTerms(S:Symbol, VARs) => S, .Patterns
  rule getGroundTerms(I:Int, VARs) => I, .Patterns
  rule getGroundTerms(X:Variable, VARs) => X, .Patterns requires notBool X in VARs
  rule getGroundTerms(X:Variable, VARs) =>    .Patterns requires         X in VARs

  rule getGroundTerms(\implies(LHS, RHS), VARs)
    => getGroundTerms(LHS, VARs) ++Patterns getGroundTerms(RHS, VARs)
  rule getGroundTerms(\equals(LHS, RHS), VARs)
    => getGroundTerms(LHS, VARs) ++Patterns getGroundTerms(RHS, VARs)
  rule getGroundTerms(\forall { UNIVs } P, VARs)
    => getGroundTerms(P, VARs ++Patterns UNIVs)
  rule getGroundTerms(\exists { UNIVs } P, VARs)
    => getGroundTerms(P, VARs ++Patterns UNIVs)
  rule getGroundTerms(\and(.Patterns), VARs)
    => .Patterns
  rule getGroundTerms(\and(P, Ps), VARs)
    => getGroundTerms(P, VARs) ++Patterns getGroundTerms(\and(Ps), VARs)
  rule getGroundTerms(\or(.Patterns), VARs)
    => .Patterns
  rule getGroundTerms(\or(P, Ps), VARs)
    => getGroundTerms(P, VARs) ++Patterns getGroundTerms(\or(Ps), VARs)
  rule getGroundTerms(\not(P), VARs)
    => getGroundTerms(P, VARs)
  rule getGroundTerms(\functionalPattern(P), VARs)
    => getGroundTerms(P, VARs)
  rule getGroundTerms(S:Symbol(ARGS:Patterns) #as APPLY, VARs)
    => APPLY , getGroundTerms(\and(ARGS))
    requires VARs -Patterns getFreeVariables(ARGS) ==K VARs
  rule getGroundTerms(S:Symbol(ARGS:Patterns) #as APPLY, VARs)
    => getGroundTerms(\and(ARGS))
    requires VARs -Patterns getFreeVariables(ARGS) =/=K VARs
```

```k
  syntax Sort ::= getSortForVariableName(VariableName, Patterns) [function]
  rule getSortForVariableName(VNAME,  VNAME  { SORT }, Vs) => SORT
  rule getSortForVariableName(VNAME1, VNAME2 { SORT }, Vs) => getSortForVariableName(VNAME1, Vs)
    requires VNAME1 =/=K VNAME2

  syntax Bool ::= isClosed(Pattern) [function]
  rule isClosed(P) => getFreeVariables(P) ==K .Patterns

  syntax Patterns ::= getFreeVariables(Patterns) [function]
  rule getFreeVariables(.Patterns) => .Patterns
  rule getFreeVariables(P, Ps)
    => removeDuplicates(getFreeVariables(P, .Patterns) ++Patterns getFreeVariables(Ps))
    requires Ps =/=K .Patterns

  rule getFreeVariables(N:Int, .Patterns) => .Patterns
  rule getFreeVariables(X:Variable, .Patterns) => X, .Patterns
  rule getFreeVariables(X:SetVariable, .Patterns) => X, .Patterns
  rule getFreeVariables(S:Symbol, .Patterns) => .Patterns
  rule getFreeVariables(S:Symbol(ARGS) , .Patterns) => getFreeVariables(ARGS)

  rule getFreeVariables(\top(), .Patterns) => .Patterns
  rule getFreeVariables(\bottom(), .Patterns) => .Patterns
  rule getFreeVariables(\equals(P1, P2), .Patterns) => getFreeVariables(P1, P2, .Patterns)
  rule getFreeVariables(\not(P), .Patterns) => getFreeVariables(P, .Patterns)
  rule getFreeVariables(\functionalPattern(P), .Patterns) => getFreeVariables(P, .Patterns)

  rule getFreeVariables(\implies(LHS, RHS), .Patterns) => getFreeVariables(LHS, RHS, .Patterns)
  rule getFreeVariables(\iff-lfp(LHS, RHS), .Patterns) => getFreeVariables(LHS, RHS, .Patterns)
  rule getFreeVariables(\and(Ps), .Patterns) => getFreeVariables(Ps)
  rule getFreeVariables(\or(Ps),  .Patterns) => getFreeVariables(Ps)
  rule getFreeVariables(\member(P1, P2))
    => getFreeVariables(P1) ++Patterns getFreeVariables(P2)
  rule getFreeVariables(\subseteq(P1, P2))
    => getFreeVariables(P1) ++Patterns getFreeVariables(P2)

  rule getFreeVariables(\exists { Vs } P,  .Patterns)
    => getFreeVariables(P, .Patterns) -Patterns Vs
  rule getFreeVariables(\forall { Vs } P,  .Patterns)
    => getFreeVariables(P, .Patterns) -Patterns Vs
  rule getFreeVariables(\mu X . P,         .Patterns)
    => getFreeVariables(P, .Patterns) -Patterns (X, .Patterns)
  rule getFreeVariables(\nu X . P,         .Patterns)
    => getFreeVariables(P, .Patterns) -Patterns (X, .Patterns)

  rule getFreeVariables(implicationContext(CONTEXT, P), .Patterns)
    => (getFreeVariables(CONTEXT, .Patterns) ++Patterns getFreeVariables(P, .Patterns))
       -Patterns #hole { Heap }, #hole { Bool }, #hole { TopSort }
  rule getFreeVariables(\typeof(P, _))
    => getFreeVariables(P)

  syntax Patterns ::= filterSetVariables(Patterns) [function]
  rule filterSetVariables(.Patterns) => .Patterns
  rule filterSetVariables(V:Variable, Vs) => V, filterSetVariables(Vs)
  rule filterSetVariables(X:SetVariable, Vs) => filterSetVariables(Vs)

// TODO: These seem specific to implication. Perhaps they need better names?
  syntax Patterns ::= getUniversalVariables(Pattern) [function]
  rule getUniversalVariables(GOAL) => getFreeVariables(GOAL, .Patterns)
  syntax Patterns ::= getExistentialVariables(Pattern) [function]
  rule getExistentialVariables(\implies(\and(LHS), \exists { EXISTENTIALS } \and(RHS)))
    => EXISTENTIALS
  rule getExistentialVariables(\implies(\and(LHS), \and(RHS)))
    => .Patterns
```

```k
  syntax Patterns ::= getImplicationContexts(Patterns) [function]
  rule getImplicationContexts(\forall {Vs} implicationContext(LCTX, RCTX), Ps)
    => \forall {Vs} implicationContext(LCTX, RCTX), getImplicationContexts(Ps)
  rule getImplicationContexts(S:Symbol(ARGs), Ps)
    => getImplicationContexts(Ps)
  rule getImplicationContexts(V:Variable, Ps)
    => getImplicationContexts(Ps)
  rule getImplicationContexts(.Patterns)  => .Patterns
```

Filters a list of patterns, returning the ones that are applications of the symbol:

```k
  syntax Patterns ::= filterByConstructor(Patterns, Symbol) [function]
  rule filterByConstructor(.Patterns, S) => .Patterns
  rule filterByConstructor((P:Symbol (Ps) , Rest), P)
    => (P:Symbol (Ps)), filterByConstructor(Rest, P)
  rule filterByConstructor((Q:Symbol (Qs) , Rest), P)
    => filterByConstructor(Rest, P)
    requires P =/=K Q
  rule filterByConstructor((Q, Rest), P) => filterByConstructor(Rest, P) [owise]
```

zip: Take two lists and return a map. This can be used to take a list of variables
and values, passed to K's substitute.

```k
  syntax Map ::= zip(Patterns, Patterns) [function]
  rule zip((L, Ls), (R, Rs)) => (L |-> R) zip(Ls, Rs)
  rule zip(.Patterns, .Patterns) => .Map

  syntax Map ::= removeIdentityMappings(Map) [function]
  rule removeIdentityMappings((L |-> R) REST) =>           removeIdentityMappings(REST)
    requires L ==K R
  rule removeIdentityMappings((L |-> R) REST) => (L |-> R) removeIdentityMappings(REST)
    requires L =/=K R
  rule removeIdentityMappings(.Map) => .Map
```

```k
  syntax String ::= VariableName2String(VariableName) [function, functional, hook(STRING.token2string)]
  syntax VariableName ::= String2VariableName(String) [function, functional, hook(STRING.string2token)]
  syntax VariableName ::= freshVariableName(Int) [freshGenerator, function, functional]
  rule freshVariableName(I:Int) => String2VariableName("F" +String Int2String(I))

  syntax SetVariable ::= String2SetVariable(String) [function, functional, hook(STRING.string2token)]
  syntax SetVariable ::= freshSetVariable(Int) [freshGenerator, function, functional]
  rule freshSetVariable(I:Int) => String2SetVariable("#F" +String Int2String(I))

  syntax Map ::= makeFreshSubstitution(Patterns) [function] // Variables
  rule makeFreshSubstitution(V { SORT }, REST)
    => V:VariableName { SORT } |-> !V1:VariableName { SORT }
       makeFreshSubstitution(REST)
  rule makeFreshSubstitution(.Patterns)
    => .Map

  syntax Patterns ::= makeFreshVariables(Patterns) [function]
  rule makeFreshVariables(P, REST) => !V1:VariableName { getReturnSort(P) }, makeFreshVariables(REST)
	requires isVariable(P)
  rule makeFreshVariables(P, REST) => !V1:SetVariable, makeFreshVariables(REST)
	requires isSetVariable(P)

  rule makeFreshVariables(.Patterns) => .Patterns
```

```k
  syntax Pattern ::= getMember(Int, Patterns) [function]
  rule getMember(0, (P:Pattern, Ps)) => P
  rule getMember(N, (P:Pattern, Ps)) => getMember(N -Int 1, Ps)
    requires N >Int 0

  syntax Patterns ::= getMembers(Ints, Patterns) [function]
  rule getMembers((I, Is), Ps) => getMember(I, Ps), getMembers(Is, Ps)
  rule getMembers(.Ints, Ps) => .Patterns

  syntax Int ::= getLength(Patterns) [function]
  rule getLength(.Patterns) => 0
  rule getLength(P, Ps) => 1 +Int getLength(Ps)

  syntax Pattern ::= getLast(Patterns) [function]
  rule getLast(Ps) => getMember(getLength(Ps) -Int 1, Ps)

  syntax Patterns ::= takeFirst(Int, Patterns) [function]
  rule takeFirst(0, _) => .Patterns
  rule takeFirst(N, (P, Ps)) => P, takeFirst(N -Int 1, Ps)
    requires N >Int 0

  syntax Patterns ::= skipFirst(Int, Patterns) [function]
  rule skipFirst(0, Ps) => Ps
  rule skipFirst(N, (P, Ps)) => skipFirst(N -Int 1, Ps)
    requires N >Int 0

  syntax Patterns ::= insertToPatterns(Int, Pattern, Patterns) [function]
  rule insertToPatterns(0, P, Ps) => (P, Ps)
  rule insertToPatterns(N, P, (P', Ps))
    => (P', insertToPatterns(N -Int 1, P, Ps))
    requires N >=Int 1

  syntax Set ::= PatternsToVariableNameSet(Patterns) [function]
  rule PatternsToVariableNameSet(.Patterns) => .Set
  rule PatternsToVariableNameSet(N{_}, Ps)
       => SetItem(N) PatternsToVariableNameSet(Ps)
```

Capture-free substitution: Substitute term or variable.
-------------------------------------------------------

TODO: This allows us to substitute arbitary terms (and not just variables) for
terms. This is very non-standard. This is currently needed because the various
unfolding strategies use this function. They should instead generate a context,
where the term being unfolded has been replace by `#hole`.

TODO: These need to use alpha equivalence rather than `==K`

```k
  syntax PatternOrVarName ::= Pattern | VariableName
  syntax Pattern ::= subst(Pattern, PatternOrVarName, Pattern)    [function, klabel(subst)]
  rule subst(T,T,V) => V // We allow substitution over arbitary patterns
  rule subst(X{_}, X:VariableName, V) => V
  rule subst(X{S}, Y:VariableName, V) => X{S} requires X =/=K Y
  rule subst(X:Variable,Y:Variable,V) => X    requires X =/=K Y
  rule subst(X:SetVariable,Y:SetVariable,V) => X requires X =/=K Y
  rule subst(X:Variable,P:Pattern, V) => X    requires notBool(isVariable(P) orBool isVariableName(P))
  rule subst(X:SetVariable,P:Pattern, V) => X
    requires notBool(isSetVariable(P))
  rule subst(I:Int, X, V) => I
  rule subst(\top(),_,_)=> \top()
  rule subst(\bottom(),_,_) => \bottom()
  rule subst(\typeof(T, S), X, V) => \typeof(subst(T, X, V), S)
  rule subst(\equals(ARG1, ARG2):Pattern, X, V)
    => \equals(subst(ARG1, X, V), subst(ARG2, X, V)):Pattern
  rule subst(\not(ARG):Pattern, X, V) => \not(subst(ARG, X, V)):Pattern
  rule subst(\and(ARG):Pattern, X, V) => \and(substPatternsMap(ARG, X |-> V)):Pattern
  rule subst(\or(ARG):Pattern, X, V)  => \or(substPatternsMap(ARG, X |-> V)):Pattern
  rule subst(\implies(LHS, RHS):Pattern, X, V)
    => \implies(subst(LHS, X, V), subst(RHS, X, V)):Pattern
  rule subst(\member(LHS, RHS):Pattern, X, V)
    => \member(subst(LHS, X, V), subst(RHS, X, V))
  rule subst(\subseteq(LHS, RHS):Pattern, X, V)
    => \subseteq(subst(LHS, X, V), subst(RHS, X, V))
  rule subst(\forall { E } C, X, V) => \forall { E } C
    requires X in E orBool X in PatternsToVariableNameSet(E)
  rule subst(\forall { E } C, X, V) => \forall { E } subst(C, X, V)
    requires notBool( X in E orBool X in PatternsToVariableNameSet(E) )
  rule subst(\exists { E } C, X, V) => \exists { E } C
    requires X in E orBool X in PatternsToVariableNameSet(E)
  rule subst(\exists { E } C, X, V) => \exists { E } subst(C, X, V)
    requires notBool( X in E orBool X in PatternsToVariableNameSet(E) )
  rule subst(\mu Y . P, X, V) => \mu Y . subst(P, X, V)
    requires notBool( X ==K Y orBool X ==K \mu Y . P)
  rule subst(\nu Y . P, X, V) => \nu Y . subst(P, X, V)
    requires notBool( X ==K Y orBool X ==K \nu Y . P)
  rule subst(S:Symbol, X, V) => S
    requires S =/=K X
  rule subst(S:Symbol(ARGS:Patterns) #as T:Pattern, X, V)
    => S(substPatternsMap(ARGS, X |-> V))
    requires T =/=K X
  rule subst(implicationContext(CTX, RHS), X, V)
    => implicationContext(subst(CTX,X,V), subst(RHS,X,V)):Pattern

  syntax Pattern ::= Pattern "[" Map "]"    [function, klabel(substMap)]
  syntax Pattern ::= substMap(Pattern, Map) [function, klabel(substMap)]
  rule substMap(BP, M) => #fun(KVPAIR => #fun(FRESHs =>
         substUnsafe( substUnsafe(BP, zip(fst(KVPAIR), FRESHs))
                    , zip(FRESHs, snd(KVPAIR))
                    )
         )( makeFreshVariables(fst(KVPAIR)))
         )( unzip(M) )

  // Renames variables incrementally not simultaneously. Helper for substMap
  syntax Pattern ::= substUnsafe(Pattern, Map) [function, klabel(substUnsafe)]
  rule substUnsafe(BP, ((X |-> V):Map REST))
    => substUnsafe(subst(BP,X,V), REST:Map)
  rule substUnsafe(BP, .Map) => BP

  syntax Patterns ::= Patterns "[" Map "]"         [function, klabel(substPatternsMap)]
  syntax Patterns ::= substPatternsMap(Patterns, Map) [function, klabel(substPatternsMap)]
  rule substPatternsMap((BP, BPs), SUBST)
    => substUnsafe(BP, SUBST), substPatternsMap(BPs, SUBST)
  rule substPatternsMap(.Patterns, SUBST) => .Patterns

  syntax Pair ::= pair(Patterns, Patterns)
  syntax Pair ::= unzip(Map) [function]
  rule unzip(.Map) => pair(.Patterns, .Patterns)
  rule unzip((L |-> R) REST) => concatenatePair(pair((L, .Patterns), (R, .Patterns)), unzip(REST))

  syntax Patterns ::= fst(Pair) [function]
  syntax Patterns ::= snd(Pair) [function]
  rule fst(pair(L, R)) => L
  rule snd(pair(L, R)) => R

  syntax Pair ::= concatenatePair(Pair, Pair) [function]
  rule concatenatePair(pair(L1, R1), pair(L2, R2)) => pair(L1 ++Patterns L2, R1 ++Patterns R2)
```

Alpha renaming: Rename all bound variables. Free variables are left unchanged.

```k
  syntax Pattern ::= alphaRename(Pattern)  [function]
  syntax Patterns ::= alphaRenamePs(Patterns) [function]
  rule alphaRename(\forall { Fs:Patterns } P:Pattern)
    => #fun(RENAMING => \forall { Fs[RENAMING] } alphaRename(substMap(P,RENAMING))) ( makeFreshSubstitution(Fs) )
  rule alphaRename(\exists { Fs:Patterns } P:Pattern)
    => #fun(RENAMING => \exists { Fs[RENAMING] } alphaRename(substMap(P,RENAMING))) ( makeFreshSubstitution(Fs) )
  rule alphaRename(\mu X . P:Pattern) => \mu !X . alphaRename(subst(P, X, !X))
  rule alphaRename(\nu X . P:Pattern) => \nu !X . alphaRename(subst(P, X, !X))
  rule alphaRename(\equals(L, R)) => \equals(alphaRename(L), alphaRename(R))
  rule alphaRename(\not(Ps)) => \not(alphaRename(Ps))
  rule alphaRename(\functionalPattern(Ps)) => \functionalPattern(alphaRename(Ps))
  rule alphaRename(\and(Ps)) => \and(alphaRenamePs(Ps))
  rule alphaRename(\or(Ps)) => \or(alphaRenamePs(Ps))
  rule alphaRename(\implies(L,R)) => \implies(alphaRename(L), alphaRename(R))
  rule alphaRename(\member(P1, P2)) => \member(alphaRename(P1), alphaRename(P2))
  rule alphaRename(\subseteq(P1, P2)) => \subseteq(alphaRename(P1), alphaRename(P2))
  rule alphaRename(S:Symbol(ARGs)) => S(alphaRenamePs(ARGs))
  rule alphaRename(S:Symbol) => S
  rule alphaRename(V:Variable) => V
  rule alphaRename(X:SetVariable) => X
  rule alphaRename(I:Int) => I
  rule alphaRename(implicationContext(P, Qs))
    => implicationContext(alphaRename(P), alphaRename(Qs))
  rule alphaRename(\typeof(P, S)) => \typeof(alphaRename(P), S)

  rule alphaRenamePs(.Patterns) => .Patterns
  rule alphaRenamePs(P, Ps) => alphaRename(P), alphaRenamePs(Ps)
```

Simplifications

```k
  syntax Patterns ::= #not(Patterns) [function]
  rule #not(.Patterns) => .Patterns
  rule #not(P, Ps) => \not(P), #not(Ps)
```

`#flattenAssoc` flattens associative binary symbols and connectives to a
single symbol applied to multiple arguments.

```k
  syntax Pattern ::= #flattenAssoc(Pattern) [function]
  rule #flattenAssoc(\and(Ps)) => \and(#flattenAnds(Ps))
  rule #flattenAssoc(\or(Ps))  => \or(#flattenOrs(Ps))
  rule #flattenAssoc(sep(Ps))  => sep(#flattenSeps(Ps))

  syntax Patterns ::= #flattenAnds(Patterns) [function]
  rule #flattenAnds(.Patterns) => .Patterns
  rule #flattenAnds(\and(Ps1), Ps2) => #flattenAnds(Ps1) ++Patterns #flattenAnds(Ps2)
  rule #flattenAnds(\or(Ps1), Ps2) => \or(#flattenOrs(Ps1)), #flattenAnds(Ps2)
  rule #flattenAnds(sep(Ps1), Ps2) => sep(#flattenSeps(Ps1)), #flattenAnds(Ps2)
  rule #flattenAnds(P, Ps) => P, #flattenAnds(Ps) [owise]

  syntax Patterns ::= #flattenSeps(Patterns) [function]
  rule #flattenSeps(.Patterns) => .Patterns
  rule #flattenSeps(emp(.Patterns), Ps2) => #flattenSeps(Ps2)
  rule #flattenSeps(sep(Ps1), Ps2) => #flattenSeps(Ps1) ++Patterns #flattenSeps(Ps2)
  rule #flattenSeps(\and(Ps1), Ps2) => \and(#flattenAnds(Ps1)), #flattenSeps(Ps2)
  rule #flattenSeps(\or(Ps1), Ps2) => \or(#flattenOrs(Ps1)), #flattenSeps(Ps2)
  rule #flattenSeps(P, Ps) => P, #flattenSeps(Ps) [owise]

  syntax Patterns ::= #flattenOrs(Patterns) [function]
  rule #flattenOrs(.Patterns) => .Patterns
  rule #flattenOrs(\or(Ps1), Ps2) => #flattenOrs(Ps1) ++Patterns #flattenOrs(Ps2)
  rule #flattenOrs(\and(Ps1), Ps2) => \and(#flattenAnds(Ps1)), #flattenOrs(Ps2)
  rule #flattenOrs(sep(Ps1), Ps2) => sep(#flattenSeps(Ps1)), #flattenOrs(Ps2)
  rule #flattenOrs(P, Ps) => P, #flattenAnds(Ps) [owise]
```

`#nnf` converts a formula into Negation Normal Form:

```k
  syntax Patterns ::= #nnfPs(Patterns) [function]
  rule #nnfPs(.Patterns) => .Patterns
  rule #nnfPs(P, Ps) => #nnf(P), #nnfPs(Ps)

  syntax Pattern  ::= #nnf(Pattern) [function]
  rule #nnf(P) => P requires isDnfAtom(P)
  rule #nnf(S:Symbol(Args)) => S(#nnfPs(Args))
  rule #nnf( \or(Ps)) =>  \or(#nnfPs(Ps))
  rule #nnf(\and(Ps)) => \and(#nnfPs(Ps))
  rule #nnf(\implies(L, R)) => #nnf(\or(\not(L), R))
  rule #nnf(\exists { .Patterns } P) => #nnf(P)

  rule #nnf(\not(P)) => \not(P) requires isDnfAtom(P)
  rule #nnf(\not(S:Symbol(Args))) => \not(S(#nnfPs(Args)))
  rule #nnf(\not( \or(Ps))) => \and(#nnfPs(#not(Ps)))
  rule #nnf(\not(\and(Ps))) => \or(#nnfPs(#not(Ps)))
  rule #nnf(\not(\not(P))) => #nnf(P)
  rule #nnf(\not(\implies(L, R))) => #nnf(\not(\or(\not(L), R)))
  rule #nnf(\not(\exists { .Patterns } P)) => #nnf(\not(P))

  syntax Bool ::= isDnfAtom(Pattern) [function]
  rule isDnfAtom(V:Variable) => true
  rule isDnfAtom(X:SetVariable) => true
  rule isDnfAtom(S:Symbol) => true
  rule isDnfAtom(\equals(L, R)) => true
  rule isDnfAtom(\exists{Vs}_) => true requires Vs =/=K .Patterns
  rule isDnfAtom(\exists{.Patterns}P) => isDnfAtom(P)
  rule isDnfAtom(\forall{Vs}_) => true
  rule isDnfAtom(\mu X . _) => true
  rule isDnfAtom(\nu X . _) => true
  rule isDnfAtom(implicationContext(_, _)) => true
  rule isDnfAtom(\and(_)) => false
  rule isDnfAtom(\or(_)) => false
  rule isDnfAtom(\implies(_, _)) => false
  rule isDnfAtom(S:Symbol(ARGS)) => false
  rule isDnfAtom(\not(P)) => false
```

```k
  syntax Pattern ::= #dnf(Pattern) [function]
  rule #dnf(P) => #flattenAssoc(#liftOr(#liftAnd(#nnf(\or(\and(P))))))
```

```k
  syntax Pattern ::= #liftOr(Pattern) [function]
  rule #liftOr(P) => \or(P) requires isDnfAtom(P)
  rule #liftOr(\not(P)) => \or(\not(P))
  rule #liftOr(S:Symbol(Args))
    => \or(#liftOr_distribute( context: S(.Patterns)
                             , processed: .Patterns
                             , remaining: Args
          )                  )
  rule #liftOr(\and(Args))
    => \or(#liftOr_distribute( context: \and(.Patterns)
                             , processed: .Patterns
                             , remaining: Args
          )                  )
  rule #liftOr(\or(Args)) => \or(#liftOrPs(Args))

  syntax Patterns ::= #liftOrPs(Patterns) [function]
  rule #liftOrPs(.Patterns) => .Patterns
  rule #liftOrPs(P, Ps) => #liftOr(P), #liftOrPs(Ps)

  syntax Patterns ::= "#liftOr_distribute" "(" "context:"   Pattern
                                           "," "processed:" Patterns
                                           "," "remaining:" Patterns
                                           ")" [function]
  rule #liftOr_distribute( context: Context
                         , processed: Processed
                         , remaining: P, Ps
                         )
    => #liftOr_distribute( context: Context
                         , processed: Processed
                         , or: #flattenAssoc(#liftOr(P))
                         , remaining: Ps
                         )
  rule #liftOr_distribute(context: S:Symbol(.Patterns), processed: Processed, remaining: .Patterns) => S(Processed), .Patterns
  rule #liftOr_distribute(context: \and(.Patterns), processed: Processed, remaining: .Patterns) => \and(Processed), .Patterns

  syntax Patterns ::= "#liftOr_distribute" "(" "context:"   Pattern
                                           "," "processed:" Patterns
                                           "," "or:"        Pattern
                                           "," "remaining:" Patterns
                                           ")" [function]
  rule #liftOr_distribute( context: Context
                         , processed: Processed
                         , or: \or(Or, Ors)
                         , remaining: Remaining
                         )
    => #liftOr_distribute( context: Context
                         , processed: Processed ++Patterns Or
                         , remaining: Remaining
                         ) ++Patterns
       #liftOr_distribute( context: Context
                         , processed: Processed
                         , or: \or(Ors)
                         , remaining: Remaining
                         )
  rule #liftOr_distribute( context: Context
                         , processed: Processed
                         , or: \or(.Patterns)
                         , remaining: Remaining
                         )
    => .Patterns 
```

```k
  syntax Pattern ::= #liftAnd(Pattern) [function]
  rule #liftAnd(P) => \and(P) requires isDnfAtom(P)
  rule #liftAnd(\not(P)) => \and(\not(P))
  rule #liftAnd(S:Symbol(Args))
    => \and(#liftAnd_distribute( context: S(.Patterns)
                               , processed: .Patterns
                               , remaining: Args
           )                   )
  rule #liftAnd(\or(Args))
    => \and(#liftAnd_distribute( context: \or(.Patterns)
                               , processed: .Patterns
                               , remaining: Args
           )                   )
  rule #liftAnd(\and(Args)) => \and(#liftAndPs(Args))

  syntax Patterns ::= #liftAndPs(Patterns) [function]
  rule #liftAndPs(.Patterns) => .Patterns
  rule #liftAndPs(P, Ps) => #liftAnd(P), #liftAndPs(Ps)

  syntax Patterns ::= "#liftAnd_distribute" "(" "context:"   Pattern
                                            "," "processed:" Patterns
                                            "," "remaining:" Patterns
                                            ")" [function]
  rule #liftAnd_distribute( context: Context
                          , processed: Processed
                          , remaining: P, Ps
                          )
    => #liftAnd_distribute( context: Context
                          , processed: Processed
                          , and: #flattenAssoc(#liftAnd(P))
                          , remaining: Ps
                          )
    requires #flattenAssoc(#liftAnd(P)) =/=K \and(.Patterns)

  rule #liftAnd_distribute( context: Context
                          , processed: Processed
                          , remaining: P, Ps
                          )
    => #liftAnd_distribute( context: Context
                          , processed: Processed ++Patterns \and(.Patterns)
                          , remaining: Ps
                          )
    requires #flattenAssoc(#liftAnd(P)) ==K \and(.Patterns)

  rule #liftAnd_distribute(context: S:Symbol(.Patterns), processed: Processed, remaining: .Patterns) => S(Processed), .Patterns
  rule #liftAnd_distribute(context: \or(.Patterns), processed: Processed, remaining: .Patterns) => \or(Processed), .Patterns

  syntax Patterns ::= "#liftAnd_distribute" "(" "context:"   Pattern
                                            "," "processed:" Patterns
                                            "," "and:"       Pattern
                                            "," "remaining:" Patterns
                                            ")" [function]
  rule #liftAnd_distribute( context: Context
                          , processed: Processed
                          , and: \and(And, Ands)
                          , remaining: Remaining
                          )
    => #liftAnd_distribute( context: Context
                          , processed: Processed ++Patterns And
                          , remaining: Remaining
                          ) ++Patterns
       #liftAnd_distribute( context: Context
                          , processed: Processed
                          , and: \and(Ands)
                          , remaining: Remaining
                          )
  rule #liftAnd_distribute( context: Context
                          , processed: Processed
                          , and: \and(.Patterns)
                          , remaining: Remaining
                          )
    => .Patterns 
```

```k
// TODO: These aren't sound
//  rule #dnfPs(\exists{Vs} P, Ps) => #exists(#dnfPs(P, .Patterns), Vs) ++Patterns #dnfPs(Ps)
//  rule #dnfPs(\forall{Vs} P, Ps) => #forall(#dnfPs(P, .Patterns), Vs) ++Patterns #dnfPs(Ps)

  syntax Patterns ::= #exists(Patterns, Patterns) [function]
  rule #exists(.Patterns, _) => .Patterns
  rule #exists((\and(Ps1), Ps2), Vs) => \exists{removeDuplicates(Vs intersect getFreeVariables(Ps1))} \and(Ps1), #exists(Ps2, Vs)
  rule #exists((\exists{Es} P, Ps2), Vs) => \exists{removeDuplicates(Es ++Patterns (Vs intersect getFreeVariables(P)))} P, #exists(Ps2, Vs)
  
  syntax Patterns ::= #forall(Patterns, Patterns) [function]
  rule #forall(.Patterns, _) => .Patterns
  rule #forall((\and(Ps1), Ps2), Vs) => \forall{removeDuplicates(Vs intersect getFreeVariables(Ps1))} \and(Ps1), #forall(Ps2, Vs)
  rule #forall((\forall{Es} P, Ps2), Vs) => \forall{removeDuplicates(Es ++Patterns (Vs intersect getFreeVariables(P)))} P, #forall(Ps2, Vs)
```

```k
  syntax Bool ::= isPredicatePattern(Pattern) [function]
  rule isPredicatePattern(\equals(_, _)) => true
  rule isPredicatePattern(\not(P)) => isPredicatePattern(P)
  rule isPredicatePattern(\functionalPattern(_)) => true
  rule isPredicatePattern(\and(.Patterns)) => true
  rule isPredicatePattern(\and(P, Ps)) => isPredicatePattern(P) andBool isPredicatePattern(\and(Ps))
  rule isPredicatePattern(\or(.Patterns)) => true
  rule isPredicatePattern(\or(P, Ps)) => isPredicatePattern(P) andBool isPredicatePattern(\or(Ps))
  rule isPredicatePattern(\implies(P1, P2)) => isPredicatePattern(P1) andBool isPredicatePattern(P2)
  rule isPredicatePattern(#hole { Bool }) => true
  rule isPredicatePattern(#hole { Heap }) => false
  rule isPredicatePattern(V:VariableName { Heap }) => false
  rule isPredicatePattern(V:SetVariable) => false

  // TODO: This should use an axiom, similar to `functional` instead: `axiom predicate(P)`
  rule isPredicatePattern(S:Symbol(ARGS)) => true
    requires getReturnSort(S(ARGS)) ==K Bool

  rule isPredicatePattern(S:Symbol(ARGS)) => false
    requires getReturnSort(S(ARGS)) =/=K Bool
  rule isPredicatePattern(emp(.Patterns)) => false
  rule isPredicatePattern(\exists{Vs} P) => isPredicatePattern(P)
  rule isPredicatePattern(\forall{Vs} P) => isPredicatePattern(P)
  rule isPredicatePattern(\mu X . P) => false
  rule isPredicatePattern(implicationContext(\and(sep(_),_),_)) => false
  rule isPredicatePattern(\typeof(_,_)) => true
  rule isPredicatePattern(implicationContext(_,_)) => true
    [owise]

  syntax Bool ::= isSpatialPattern(Pattern) [function]
  rule isSpatialPattern(pto(_)) => true
  rule isSpatialPattern(emp(.Patterns)) => true
  rule isSpatialPattern(sep(.Patterns)) => true
  rule isSpatialPattern(sep(P, Ps)) => isSpatialPattern(P) andBool isSpatialPattern(sep(Ps))
  rule isSpatialPattern(P) => false
    requires isPredicatePattern(P)
  rule isSpatialPattern(\and(_)) => false
  rule isSpatialPattern(\or(_)) => false
  rule isSpatialPattern(S:Symbol(ARGS)) => true
    requires S =/=K sep andBool getReturnSort(S(ARGS)) ==K Heap
  rule isSpatialPattern(S:Symbol(ARGS)) => false
    requires getReturnSort(S(ARGS)) =/=K Heap
  rule isSpatialPattern(#hole { Bool }) => false
  rule isSpatialPattern(#hole { Heap }) => true
  rule isSpatialPattern(V:VariableName { Heap }) => true
  rule isSpatialPattern(V:SetVariable) => false
  rule isSpatialPattern(\mu X . P) => false

  // TODO: Perhaps normalization should get rid of this?
  rule isSpatialPattern(\exists{_} implicationContext(\and(sep(_),_),_)) => true
  rule isSpatialPattern(\exists{_} implicationContext(_,_)) => false
    [owise]
  rule isSpatialPattern(\forall{_} implicationContext(\and(sep(_),_),_)) => true
  rule isSpatialPattern(\forall{_} implicationContext(_,_)) => false
    [owise]

  syntax Patterns ::= getSpatialPatterns(Patterns) [function]
  rule getSpatialPatterns(.Patterns) => .Patterns
  rule getSpatialPatterns(P, Ps) => P, getSpatialPatterns(Ps)
    requires isSpatialPattern(P)
  rule getSpatialPatterns(P, Ps) => getSpatialPatterns(Ps)
    requires notBool isSpatialPattern(P)

  syntax Patterns ::= getPredicatePatterns(Patterns) [function]
  rule getPredicatePatterns(.Patterns) => .Patterns
  rule getPredicatePatterns(P, Ps) => P, getPredicatePatterns(Ps)
    requires isPredicatePattern(P)
  rule getPredicatePatterns(P, Ps) => getPredicatePatterns(Ps)
    requires notBool isPredicatePattern(P)
```

```k
  syntax Bool ::= isDisjunction(Pattern) [function]
  rule isDisjunction(\or(_)) => true
  rule isDisjunction(_) => false
    [owise]

  syntax Bool ::= isApplication(Pattern) [function]
  rule isApplication(S:Symbol(_)) => true
  rule isApplication(_) => false
    [owise]

  syntax Int ::= lengthPatterns(Patterns) [function]
  rule lengthPatterns(.Patterns) => 0
  rule lengthPatterns(P, Ps) => 1 +Int lengthPatterns(Ps)

  syntax Bool ::= hasImplicationContext(Pattern)  [function]
  syntax Bool ::= hasImplicationContextPs(Patterns)  [function]
  rule hasImplicationContext(X:Variable) => false
  rule hasImplicationContext(X:SetVariable) => false
  rule hasImplicationContext(X:Int) => false
  rule hasImplicationContext(S:Symbol) => false
  rule hasImplicationContext(\implies(LHS, RHS))
    => hasImplicationContext(LHS) orBool hasImplicationContext(RHS)
  rule hasImplicationContext(\equals(LHS, RHS))
    => hasImplicationContext(LHS) orBool hasImplicationContext(RHS)
  rule hasImplicationContext(S:Symbol (ARGS)) => hasImplicationContextPs(ARGS)
  rule hasImplicationContext(\and(Ps)) => hasImplicationContextPs(Ps)
  rule hasImplicationContext(\or(Ps)) => hasImplicationContextPs(Ps)
  rule hasImplicationContext(\not(P)) => hasImplicationContext(P)
  rule hasImplicationContext(\functionalPattern(P)) => hasImplicationContext(P)
  rule hasImplicationContext(\exists{ _ } P ) => hasImplicationContext(P)
  rule hasImplicationContext(\forall{ _ } P ) => hasImplicationContext(P)
  rule hasImplicationContext(\mu X . P) => hasImplicationContext(P)
  rule hasImplicationContext(implicationContext(_, _)) => true
  rule hasImplicationContextPs(.Patterns) => false
  rule hasImplicationContextPs(P, Ps)
    => hasImplicationContext(P) orBool hasImplicationContextPs(Ps)
  rule hasImplicationContext(\typeof(P, _))
    => hasImplicationContext(P)

  syntax Pattern ::= "maybeExists" "{" Patterns "}" Pattern [function]
  rule maybeExists{.Patterns} P => P
  rule maybeExists{V, Vs} P => \exists{V, Vs} P

  syntax Pattern ::= "maybeAnd" "(" Patterns ")" [function]
  rule maybeAnd(P, .Patterns) => P
  rule maybeAnd(Ps) => \and(Ps) [owise]

  syntax String ::= AxiomNameToString(AxiomName) [function, hook(STRING.token2string)]
  syntax AxiomName ::= StringToAxiomName(String) [function, functional, hook(STRING.string2token)]
                     | freshAxiomName(Int)       [freshGenerator, function, functional]

  rule freshAxiomName(I:Int) => StringToAxiomName("ax" +String Int2String(I))

  syntax String ::= ClaimNameToString(ClaimName) [function, hook(STRING.token2string)]
  syntax ClaimName ::= StringToClaimName(String) [function, functional, hook(STRING.string2token)]
                     | freshClaimName(Int)       [freshGenerator, function, functional]

  rule freshClaimName(I:Int) => StringToClaimName("cl" +String Int2String(I))

  syntax Set ::= collectClaimNames() [function]
               | #collectClaimNames(Set) [function]

  rule collectClaimNames() => #collectClaimNames(.Set)

  rule [[ #collectClaimNames(Ns)
          => #collectClaimNames(Ns SetItem(ClaimNameToString(N))) ]]
       <id> N:ClaimName </id>
       requires notBool (ClaimNameToString(N) in Ns)

  rule #collectClaimNames(Ns) => Ns [owise]

  syntax Declarations
    ::= collectDeclarations(GoalId) [function]
    | collectLocalDeclarations(GoalId) [function]
    | #collectLocalDeclarations(GoalId, Declarations) [function]

  rule collectDeclarations(GId)
       => collectGlobalDeclarations() ++Declarations
          collectLocalDeclarations(GId)

  syntax Declarations
  ::= collectLocalDeclarations(GoalId) [function]
    | #collectLocalDeclarations(GoalId, Declarations) [function]

  rule collectLocalDeclarations(GId)
       => #collectLocalDeclarations(GId, .Declarations)

  rule [[ #collectLocalDeclarations(GId, Ds)
       => #collectLocalDeclarations(GId, D Ds) ]]
    <id> GId </id>
    <local-decl> D </local-decl>
    requires notBool (D inDecls Ds)

  rule #collectLocalDeclarations(_, Ds) => Ds [owise]

  syntax Declarations ::= collectGlobalDeclarations() [function]
                        | #collectGlobalDeclarations(Declarations) [function]
                        | #collectSortDeclarations(Declarations) [function]

  rule collectGlobalDeclarations() => #collectGlobalDeclarations(.Declarations)

  rule [[ #collectGlobalDeclarations(Ds) => #collectGlobalDeclarations(D Ds) ]]
    <declaration> D </declaration>
    requires notBool (D inDecls Ds) andBool notBool isSortDeclaration(D)

  // We need to gather sort declarations last so sorts are declared correctly
  // when translating to smt
  // TODO: do we need to gather symbol decs last as well?
  rule [[ #collectSortDeclarations(Ds) => #collectSortDeclarations(D Ds) ]]
    <declaration> (sort _ #as D:Declaration) </declaration>
    requires notBool (D inDecls Ds)

  rule #collectGlobalDeclarations(Ds) => #collectSortDeclarations(Ds) [owise]
  rule #collectSortDeclarations(Ds) => Ds [owise]

  syntax Bool ::= Declaration "inDecls" Declarations [function]
  rule _ inDecls .Declarations => false
  rule D inDecls D Ds => true
  rule D inDecls D' Ds => D inDecls Ds
    requires D =/=K D'
```
Functions `getUniversallyQuantifiedVariables` and `getConclusion`
assume a pattern of the form:
  `\forall{Vs1} A1 -> \forall{Vs2} A2 -> ... -> C`
```k
  syntax Patterns ::= getUniversallyQuantifiedVariables(Pattern) [function]
  rule getUniversallyQuantifiedVariables(_) => .Patterns [owise]
  rule getUniversallyQuantifiedVariables(\implies(_, P))
       => getUniversallyQuantifiedVariables(P)
  rule getUniversallyQuantifiedVariables(\forall{Vs} P)
       => Vs ++Patterns getUniversallyQuantifiedVariables(P)

  syntax Pattern ::= getConclusion(Pattern) [function]

  rule getConclusion(\forall{_} P)
       => getConclusion(P)
  rule getConclusion(\implies(_,P))
       => getConclusion(P)
  rule getConclusion(P) => P [owise]

  // <public>
  syntax Patterns ::= getSetVariables(Pattern) [function]
  // </public>
  // <private>
  syntax Patterns ::= getSetVariablesVisitorResult(VisitorResult) [function]
  syntax Visitor ::= getSetVariablesVisitor(Patterns)
  rule getSetVariables(P)
    => getSetVariablesVisitorResult(
         visitTopDown(
           getSetVariablesVisitor(.Patterns),
           P
         )
       )

  rule getSetVariablesVisitorResult(
         visitorResult(getSetVariablesVisitor(Ps), _))
    => Ps

  rule visit(getSetVariablesVisitor(Ps), P) => visitorResult(getSetVariablesVisitor(P, Ps), P) requires         isSetVariable(P)
  rule visit(getSetVariablesVisitor(Ps), P) => visitorResult(getSetVariablesVisitor(   Ps), P) requires notBool isSetVariable(P)


  // </private>

  syntax String ::= getFreshName(String, Set) [function]
                  | getFreshNameNonum(String, Set) [function]
                  | #getFreshName(String, Int, Set) [function]

  rule getFreshName(Prefix, S) => #getFreshName(Prefix, 0, S)
  rule #getFreshName(Prefix, N => N +Int 1, S)
       requires (Prefix +String Int2String(N)) in S
  rule #getFreshName(Prefix, N, S) => Prefix +String Int2String(N)
       requires (notBool ((Prefix +String Int2String(N)) in S))

  rule getFreshNameNonum(Prefix, S)
       => #if Prefix in S #then
            getFreshName(Prefix, S)
          #else
            Prefix
          #fi

  syntax Set ::= collectGlobalAxiomNames() [function]
               | collectLocalAxiomNames(GoalId) [function]
               | #declarationsToAxiomNames(Declarations) [function]

  rule collectGlobalAxiomNames()
    => #declarationsToAxiomNames(collectGlobalDeclarations())
  rule collectLocalAxiomNames(GId)
    => #declarationsToAxiomNames(collectLocalDeclarations(GId))
  rule #declarationsToAxiomNames(.Declarations) => .Set
  rule #declarationsToAxiomNames((axiom N : _) Ds)
       => SetItem(AxiomNameToString(N)) #declarationsToAxiomNames(Ds)
  rule #declarationsToAxiomNames(D Ds => Ds) [owise]


  syntax Set ::= collectGlobalNamed() [function]
  rule collectGlobalNamed()
    => collectGlobalAxiomNames() collectClaimNames()

  syntax Set ::= collectNamed(GoalId) [function]
  rule collectNamed(GId)
    => collectGlobalNamed() collectLocalAxiomNames(GId)

  syntax AxiomName ::= getFreshGlobalAxiomName() [function]
  rule getFreshGlobalAxiomName()
       => StringToAxiomName(getFreshName("ax", collectGlobalNamed()))

  syntax AxiomName ::= getFreshAxiomName(GoalId) [function]
  rule getFreshAxiomName(GId)
       => StringToAxiomName(getFreshName("ax", collectNamed(GId)))

  syntax ClaimName ::= getFreshClaimName() [function]
  rule getFreshClaimName()
       => StringToClaimName(getFreshName("cl", collectGlobalNamed()))

  syntax Set ::= collectSymbolsS(GoalId) [function]
               | #collectSymbolsS(Declarations) [function]

  rule collectSymbolsS(GId)
       => #collectSymbolsS(collectDeclarations(GId))

  rule #collectSymbolsS(.Declarations) => .Set
  rule #collectSymbolsS( (symbol S ( _ ) : _) Ds)
       => SetItem(SymbolToString(S)) #collectSymbolsS(Ds)
  rule #collectSymbolsS(_ Ds) => #collectSymbolsS(Ds) [owise]

  syntax Symbol ::= StringToSymbol(String) [function, functional, hook(STRING.string2token)]

  syntax Symbol ::= getFreshSymbol(GoalId, String) [function]
  rule getFreshSymbol(GId, Base)
       => StringToSymbol(
            getFreshNameNonum(Base, collectSymbolsS(GId)))

  syntax Patterns ::= filterVariablesBySort(Patterns, Sort) [function]
  rule filterVariablesBySort(.Patterns, _) => .Patterns
  rule filterVariablesBySort(((_ { S } #as V), Vs), S)
    => V, filterVariablesBySort(Vs, S)
  rule filterVariablesBySort((V, Vs), S)
    => filterVariablesBySort(Vs, S) [owise]
```

```k
endmodule
```
