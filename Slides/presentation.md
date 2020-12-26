---
title: Formal Verification of Authenticated, Append-Only, Skip Lists in Agda
author: Victor Cacciari Miraldo, Harold Carr, Mark Moir, Lisandra Silva and Guy L. Steele Jr.
header-includes: |
  \usepackage{tikz}
  \usetikzlibrary{positioning}
  \usetikzlibrary{calc}
---

\newcommand{\hash}{\mathrm{hash}}

# Traditional Append-Only Structures: Blockchains

- Only _lookup_ and _append_ operations\pause
- Block $n+1$ depends on block $n$
\vfill
\begin{tikzpicture}
	\node (gen) {$\star$};
	\pause
	\node [right = of gen] (e1) {$e_1$};
	\draw[->] (e1.north west) to[out=135, in=45] (gen.north east);
	\pause
	\node [right = of e1] (e2) {$e_2$};
	\draw[->] (e2.north west) to[out=135, in=45] (e1.north east);
	\pause
	\node [right = of e2] (elipsis) {$\cdots$};
	\draw[->] (elipsis.north west) to[out=135, in=45] (e2.north east);
	\node [right = of elipsis] (en) {$e_n$};
	\draw[->] (en.north west) to[out=135, in=45] (elipsis.north east);
	\node [right = of en] (en1) {$e_{n+1}$};
	\draw[->] (en1.north west) to[out=135, in=45] (en.north east);
\onslide<1->
\end{tikzpicture}
\vfill
\pause
- Prevent attacker rewriting history:
  + We agree on $\hash\;e_{n+1}$ implies we agree on $\hash\;e_n$\pause

- Works well with static membership and no garbage collection

\vfill

# Traditional Append-Only Structures: Blockchains

```haskell
data Auth a = Auth a Digest

type Log a = [Auth a]

mkauth :: (Hashable a) => a -> Log a -> Auth a
mkauth x []              = error "Log not initialized"
mkauth x (Auth y dy : l) = Auth x (hash (hash x ++ dy))

append :: (Hashable a) => a -> Log a -> Log a
append x l = mkauth x l : l
```

# Problem: Dynamic Participation

\vfill
\begin{tikzpicture}
	\node (gen) {$\star$};
	\node [right = of gen] (e1) {$e_1$};
	\draw[->] (e1.north west) to[out=135, in=45] (gen.north east);
	\node [right = of e1] (e2) {$e_2$};
	\draw[->] (e2.north west) to[out=135, in=45] (e1.north east);
	\node [right = of e2] (elipsis) {$\cdots$};
	\draw[->] (elipsis.north west) to[out=135, in=45] (e2.north east);
	\node [right = of elipsis] (en) {$e_n$};
	\draw[->] (en.north west) to[out=135, in=45] (elipsis.north east);
	\node [right = of en] (en1) {$e_{n+1}$};
	\draw[->] (en1.north west) to[out=135, in=45] (en.north east);
	\pause
	\node [below = of en1] (p) {Participant $p$ joins};
	\draw[->] (p) -- (en1);
\onslide<1->
\end{tikzpicture}
\vfill
\pause

- New participant needs either:
  1. Download the entire history and compute state $s_{n+1}$\pause
  1. Download a "checkpoint" package
	 + copy of $s_{n+1}$
	 + enough signatures over $\hash\;e_{n+1}$\pause

- Option (2) is better: log always increases
\vfill

# Dynamic Participation: Verifying Claims

Say $p$ began participation at $n$, \pause now say $p$ receives a claim about some
entry $e_i$, for $i < n$. How should $p$ check the claim's veracity?

\vfill

1. Download entries between $i$ and $n$; rebuild the hashes; check
rebuild hash for $e_n$ against known one
1. Forward the claim to some other participant

\vfill

After a few rounds of dynamic participation no participant might contain all entries.

\vfill

# Append-Only Authenticated Skip Lists (AAOSL)

- Originally from Baker and Maniatis ([arxiv pdf](http://arxiv.org/abs/cs/0302010))
- Make $\hash\;e_n$ depend on more than one entry.\pause
  + Let $n = 2^l * d$, for an odd $d$, $\hash\;e_d$ will depend
  on $e_i$, $i \in \{ e_{2^l*d - 2^k} \mid k \leq l \}$.\pause

\resizebox{\textwidth}{!}{\begin{tikzpicture}
	\node (gen) {$\star$};
	\node [right = of gen] (e1) {$e_1$};
	\draw[->] (e1.north west) to[out=135, in=45] (gen.north east);
	\node [right = of e1] (e2) {$e_2$};
	\draw[->] (e2.north west) to[out=135, in=45] (e1.north east);
	\node [right = of e2] (e3) {$e_3$};
	\draw[->] (e3.north west) to[out=135, in=45] (e2.north east);
	\node [right = of e3] (e4) {$e_4$};
	\draw[->] (e4.north west) to[out=135, in=45] (e3.north east);
	\node [right = of e4] (e5) {$e_5$};
	\draw[->] (e5.north west) to[out=135, in=45] (e4.north east);
	\node [right = of e5] (e6) {$e_6$};
	\draw[->] (e6.north west) to[out=135, in=45] (e5.north east);
	\node [right = of e6] (e7) {$e_7$};
	\draw[->] (e7.north west) to[out=135, in=45] (e6.north east);
	\node [right = of e7] (e8) {$e_8$};
	\draw[->] (e8.north west) to[out=135, in=45] (e7.north east);
	\node [right = of e8] (e9) {$e_9$};
	\draw[->] (e9.north west) to[out=135, in=45] (e8.north east);
	\draw[->] ($ (e2.north west) + (.1,0) $) to[out=110,in=70] ($ (gen.north east) - (.1,0) $);
	\draw[->] ($ (e4.north west) + (.1,0) $) to[out=110,in=70] ($ (e2.north east) - (.1,0) $);
	\draw[->] ($ (e6.north west) + (.1,0) $) to[out=110,in=70] ($ (e4.north east) - (.1,0) $);
	\draw[->] ($ (e8.north west) + (.1,0) $) to[out=110,in=70] ($ (e6.north east) - (.1,0) $);
	\pause
	\draw[->] ($ (e4.north west) + (.2,0) $) to[out=105,in=75] ($ (gen.north east) - (.2,0) $);
	\draw[->] ($ (e8.north west) + (.2,0) $) to[out=105,in=75] ($ (e4.north east) - (.2,0) $);
	\pause
	\draw[->] ($ (e8.north west) + (.3,0) $) to[out=100,in=90] ($ (gen.north east) - (.3,0) $);
\onslide<1->
\end{tikzpicture}}

\pause
- The _level_ of $2^l*d$ is defined as $l+1$, and indicates the number of hops _out_ of $2^l*d$.
  + i.e., index 8 has level $4$ and depends on four distinct hashes: 7,6,4 and $\star$.

# Advancement Proofs

# Membership Proofs

# The Core Security Principle(s)

# The Knapking Security Proof

# AAOSL In Agda

# Future Work and Conclusions