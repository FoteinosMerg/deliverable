% title Usage of zk-SNARKs
# Usage of zk-SNARKs

## Generic zk-SNARK scheme

### Preliminaries

A zk-SNARK (Zero-Knowledge Succinct Non-interactive ARgument of
Knowledge) is a general purpose cryptographic scheme for generating
and verifying zero-knowledge proofs in efficient and non-interactive
fashion. Statements to be proven are claims of knowledge of a quantity
*w* (called *withness*) satisfying *F(x, w)*, where *x* is a public
quantity (called *instance*) and *F* is a public relation (called
*predicate*). More accurately, without revealing info about *w*, the
prover wishes to generate a proof of the following statement:

$$
\text{given } x \in X \text{ and } F: X \times W \mapsto \{0,
        1\}, \text{ I know a } w \text{ such that } F(x, w) = 1
$$

where $X$ is the set of acceptable public instances, $W$ is the set of
acceptable private witnesses and 0, 1 hold their usual logical
meanings. For example, if the verifier were a web server holding a
database of hashed passwords and the prover were the web client of a
registered user, the latter does not need to send their password *w*
(as cleartext or in disguise) in order to login, but only a proof of
the above statement with predicate:

$$ F(x, w) := \{H(w) = x\}$$

where $H$ is the hash function used by the server, *W* being the
preimage set and $X$ the set of stored hashed passwords in the
database. To give a more relevant example from the cryptocurrency
framework, if $w$ were the encoding of the private payment details of
a particular transaction and $x$ the encrypted version of this data,
then the predicate $F$ would typically be

$$ F(x, w) = \{w \text{ is a well-formed transaction and } x
        \text{ is the encryption of it}\} $$

Note that $F$ is predefined and fixed, which allows for
*non-interactive* mode (see below). Generation and verification of
proofs for various witness-instance pairs $(w, x)$ is instead a
massively repeated procedure. Efficiency of the cryptosystem refers
thus not to its initial setup (which only involves $F$) but to the
generation and verification of proofs (involving arbitrary
instantiations of $(w, x)$ pairs), which naturally leads to the notion
of *succinctness* (see below).


### Predicate reduction

In theory, any NP-complete statement which is expressible in the above
form may serve as the fixed public predicate of a zk-SNARK
cryptosystem. However, transforming the predicate so that it be
amenable to low-level cryptographic operations is a genuine challenge.
That is, except if following recipes, configuration of the
cryptosystem may well be a non-trivial step from the implementors'
viewpoint.

After completion of the initial setup, the predicate $F$ may be
thought of as a fixed circuit of logical gates. In a blockchain
context, this means to encode in the form of a circuit the consensus
rules of the network. Satisfaction of $F$ for a witness-instance pair
$(w, x)$ emulates a particular state of the circuit after running it
with that input. zk-SNARK libraries usually avail a mechanism for
modelling predicates as logical circuits which resembles the concept of
*protoboard* (prototyping board) from electrical engineering: the
cryptosystem's configuration presupposes allocation of $F$'s "chips"
to the protoboard.

This is however only a convenient abstraction. What in fact one must
do is appropriately reduce the NP-statement $F$ to a quadratic system of
polynomial equations. Satisfiability of $F$ for a given pair $(w, x)$
amounts to $(x, w)$ being (with overwhelming probability) a solution
to this system of equations. More specifically, it has been proven that
any NP-complete statement can be reduced to such system of a special
kind, usually referred to as a *set of quadratic constraints*, with
each zk-SNARK proof attesting that this set of constraints is
satisfied by some concrete input. Further results show that any set of
quadratic constraints can be conveniently flattened to what is known
as *R1CS* (*Rank 1 Constraint System*). Predicate reduction amounts
thus to reformulating an NP-complete statement as R1CS.


### Cryptographic flow

We expose here the ZS-NARK cryptographic flow for reference, in order to
gradually attain semi-formal language and terminology. The flow involves
three basic algorithms run by homonymous and mutually independent
components: *Setup*, *Prover* and *Verifier*. How these components are
distributed among network entities depends on the protocol under
implementation.

The Setup algorithm is typically run once by a unanimously trusted
party. It is fed with a R1CS (i.e., the fixed public predicate *F*)
and outputs the public CRS (*Common Reference String*) involved in any
subsequent proof generation and verification. In particular, the CRS
consists of a proving key *pk* (uniformly used by all Provers) and a
uniquely correlated verification key *vk* (uniformly used by all
Verifiers). Computation of CRS involves also a once used randomness
$\mathit{lambda}$, so that it can formally be expressed as follows:

$$ (F, \mathit{lambda}) \mapsto \text{CRS} := (pk, vk) =
   \mathrm{Setup}(F, \mathit{lambda})$$

It is essential that $\mathit{lambda}$ remains forever secret: leakage of
randomness would allow a malicious prover to generate false proofs
that can erroneously be verified. Trustedness of the Setup party must
thus be equivalent to the unanimously accepted guarantee, that the
$\mathit{lambda}$ parameter has irreversibly been eliminated after CRS
generation (which is the reason for the destruction ceremonies
surrounding ZCash etc.).

Upon every proof generation, the Prover is fed with a public instance
$x$ and a secret witness $w$ for it. Using the proving part
$\mathit{pk}$ of the CRS, the prover generates a proof $\pi$ attesting that
$F(x, \cdot)$ is satisfiable by some known $w$, without disclosing any info
about $w$ (i.e., except for the claim that it is known). This can formally be
expressed as

$$ (\mathit{pk}, x, w) \mapsto \pi = \text{Prover}(\mathit{pk}, x, w) $$

The Verifier uses the verifying part $\mathit{vk}$ of the CRS to
operate upon the provided proof, accepting or rejecting according to
whether it was found to be valid, resp. invalid:

$$ (vk, x, \pi) \mapsto \text{Verifier}(vk, x, \pi) \in \{0, 1\} $$

Note that typically anyone can run a Verifier.

The whole flow is summarized in the following figure:

...[Figure] (1)

In this context, the zk-SNARK properties are formalized as follows:

  * *zk* (*Zero Knowledge*): The generated proof conveys no info about
  $w$ beyond the claim of it being known, i.e., that $F(x, .)$ is
  satisfied by it.

  * *S* (*Succint*): The size of proofs is small, constant and
  independent of the system's constraints and public inputs. More
  accurately, ignoring dependence on security parameters, proof
  generation is $O(1)$ and verification is $O(k)$, where $k$
  stands for the proof uniform bitsize. The bottleneck of initial
  setup needs not be taken into account.

  * *N* (*Non-interactive*): The only interaction between Prover and
  Verifier is the transmission of the produced proof (one
  communication step). Any intermediate computation required for proof
  generation and verification has been absorbed in the precomputation
  of the CRS (involved via usage of $\mathit{pk}$ and $\mathit{vk}$).

Generic requirements may also be formalized as follows:

  * *Completeness*: Correct proofs (i.e., generated upon a correct
  witness by a honest prover) always verify. More accurately,

  $$
  \begin{align*}
  &\textrm{for any } \pi =
  \mathrm{Prover}(pk, x, w) \textrm{ such that } F(x, w) = 1,\\
  &\textrm{there holds } \mathrm{Verifier}(vk, x, \pi) = 1
  \end{align*}
  $$

  * *Soundness*: No false proof (i.e., generated upon incorrect
  witness by a malicious prover) ever verifies. More accurately,

  $$
  \begin{align*}
  &\textrm{for any } x \textrm{ such that } F(x, .)
  \textrm{ not satisfiable, there exists no } w \text{ such that } \\
  &\mathrm{Verifier}(vk, x, \pi) = 1 \text{ with } \pi =
  \mathrm{Prover}(pk, x, w)
  \end{align*}
  $$

### `libsnark` library

In view of the above nomenclature, we will here try to determine
respective API calls to the `libsnark` library. Note that in
the Libsnark unofficial documentation and terminology, the Setup
component is referred to as *generator*, public instances as
*primary inputs* and secret witnesses as *auxiliary inputs*.

Libsnark provides a class `protoboard` for generating CRSs upon
arbitrary R1CS circuits, i.e., allowing for great configurability with
respect to the public predicate. This is crucial for our use case,
since the consensus rules to be modelled are not precisely known in
advance. It should be nevertheless mentioned that Libsnark provides
also a series of *gadgets*, i.e., ready made wrappers around
protoboards that automatically handle R1CS and witness generation for
a broad range of special cases. These functionalities are the
``generate_r1cs_constraints()`` and ``generate_r1cs_witness()`` public
methods of gadget objects respectively.

### `libsnark` API calls

#### R1CS generation

We first need to initialize elliptic curve parameters:

```c
default_r1cs_ppzksnark_pp::init_public_params();
typedef libff::Fr<default_r1cs_ppzksnark_pp> FieldT;
```

Protoboard creation and allocation of variables upon it proceeds as
follows:

```c
protoboard<FieldT> pb;

pb_variable<FieldT> var_1;
pb_variable<FieldT> var_2;
...

var_1.allocate(pb, "var_1");
var_2.allocate(pb, "var_2");
...
```

We can then specify that the first *n* variables constitute primary
input (i.e., public instance) as follows:

```c
pb.set_input_sizes(n);
```

in which case the last $m - n$ variables will correspond to
auxiliary input (i.e., secret witness). It is then straightforward to
impose quadratic constraints on the above allocated variables by means
of the `add_r1cs_constraint()` method. For example, a constraint
`var_1 ^ 2 = var_3` would be enforced as follows:

```c
pb.add_r1cs_constraint(r1cs_constraint<FieldT>(var_1, var_1, var_3));
```

After imposing further constraints, the resulting circuit is finally
exported:

```c
r1cs_constraint_system<FieldT> constraint_system = pb.get_constraint_system();
```

#### Setup

Once reduced to a R1CS as above, the public predicate can be fed to
the CRS generator:

```c
r1cs_ppzksnark_keypair<ppT> keypair = r1cs_ppzksnark_generator<ppT>(r1cs_constraint_system);
```

If one wishes to make use of the Groth16 zk-SNARK protocol, the above
statement may slightly be modified as follows:

```c
r1cs_gg_ppzksnark_keypair<ppT> keypair = r1cs_gg_ppzksnark_generator<ppT>(r1cs_constraint_system);
````

In either case, the proving and verification keys are directly
accessible as the `pk` and `vk` attributes of the generated
keypair (see below).

#### Prover

Given the above CRS ``keypair``, proof generation proceeds as follows:

```c
r1cs_ppzksnark_proof<ppT> proof = r1cs_ppzksnark_prover<ppT>(keypair.pk, pb.primary_input(), pb.auxiliary_input());
```

In the Groth16 zk-SNARK protocol, the corresponding formulation would be

```c
r1cs_gg_ppzksnark_proof<ppT> proof = r1cs_gg_ppzksnark_prover<ppT>(keypair.pk, pb.primary_input(), pb.auxiliary_input());
```

#### Verifier

The above generated ``proof`` can be verified as follows:

```c
bool ans = r1cs_ppzksnark_verifier_strong_IC<ppT>(keypair.vk, pb.primary_input(), proof);
```

with the corresponding Groth16 formulation being

```c
bool ans = r1cs_gg_ppzksnark_verifier_strong_IC<ppT>(keypair.vk, pb.primary_input(), proof);
```

### Optimizing verification

For the purpose of faster verification, ``vk``
can be further processed during the initial setup, yielding the so
called *processed verification key* ``pvk``. In particular, a small
amount of extra precomputed info can be added to it, in which case the
Verifier algorithm must also be modified appropriately (*online*
Verifier). In this case, Figure 1 transforms as follows:

... [Figure] (2)

In Libsnark, preprocessing of the verification key corresponds to the
following statement:

```c
r1cs_ppzksnark_processed_verification_key<ppT> pvk = r1cs_ppzksnark_verifier_process_vk<ppT>(keypair.vk);
```

Proof verification should then proceed as follows:

```c
bool ans = r1cs_ppzksnark_online_verifier_strong_IC<ppT>(pvk, pb.primary_input(), proof);
```

the corresponding Groth16 statement being

```c
bool ans = r1cs_gg_ppzksnark_online_verifier_strong_IC<ppT>(pvk,
pb.primary_input(), proof);
```

Note that proof generation is not affected by the usage of
preprocessed verification keys.


## Application in our use case

### Predicate formulation and circuit construction

[Describe how to encode the network consensus rules as the
cryptosystem's fixed public predicate and how to reduce the latter to
a R1CS circuit]

### Role distribution among network entities

[Describe which network entities will run the algorithms *Setup* or
*Prover* or *Verifier* or combinations thereof]

### Toolkit API calls

[In view of the above role distribution, formulate API calls at the
network layer wrapping or utilizing API calls to Libsnark]
