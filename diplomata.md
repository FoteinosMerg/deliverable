# Anonymous Proof of Possession in the context of Diplomata Record Ledger (UC3)

## Introduction

*Diplomata* is a proposed system for proving and verifying title
possession in a privacy-preserving fashion. It involves three kinds of
interacting network entities: *Holders* of qualifications, which
they can present to interested parties, *Issuers* capable of issuing
qualifications to holders, and *Verifiers* able to verify the holders' claims
of possessing a title. Awarding of titles, requests of certification,
acknowledgments of certification etc. are all recorded in the form of
transaction in a blockchain ledger, in order for the
involved entities to be held publicly accountable for their actions.
Requirements like access control on behalf of the holders, resistance to
(some level of) no honesty and auditability by external observers should
also be satisfied.

A *ZK-Decryptor Protocol* (*ZKD*) has already been proposed on top of an
El-Gamal cryptosystem, that allows holders to (indirectly) prove possession of
titles to verifiers. However, the Issuer of the title is heavily involved in
the process. Except for an initial certification request and the private
communication of the accompanying commitment to the Verifier, the rest of
communication takes place between Issuer and Verifier: it is the *Issuer* who
actually commits and generates the proof of award that will be finally verified.
More importantly, verification of proof presupposes that the (content of the)
title will be decrypted by the Verifier and thus *disclosed* to them, i.e.,
"zero-knowledge" does not refer to the qualification's content. Since the title
contains as such reference to the Holder's identity, this entails
*non*-anonymity for the Holder.

Purpose of this text is to propose an alternative to the *ZKD* protocol
for the case where the Holder's anonymity is desired and the (content of the)
title should not be revealed. This pertains to the following meritocratic
scenario. Holder and Verifier meet anonymously online with the requirement that
their communication channel remains private and their identities unrevealed.
In particular, the Holder wishes to prove possession of a title $t$

1. without ever revealing their identity, since the communication must remain
anonymous
2. without disclosing any info about $t$, except for
the fact that it exists among their acquired qualifications
3. in one single communication step
4. without involvement of anybody else in that step (say, the Issuer of $t$),
since the communication must remain private

The proposed protocol will be referred to as
*Anonymous Proof of Possession Protocol* (*APP*). Proof of possession should be
thought of as proof of inclusion in a set of recordedly awarded titles
(possibly of a special kind) strongly correlated to the prover's (unrevealed)
identity. In doing so, we will employ the *zk-SNARK* cryptographic
scheme combined with a commitment mechanism. In particular, proof of possession
will be a zk-SNARK proof.

Such a construction will partly resemble the *ZeroCash*
anonymous payment scheme, with each title interpreted as 1 BTC (bitcoin) and
the holders' commitments recorded in the ledger.
There are, however, two major differences:

1. No "Direct Anonymous Payment" method is here facilitated, that is, transfer
of titles(=coins) from holder to holder is not anyhow taken into account.
This allows to stay away from the intricacies of the key-address machinery
surrounding commitments in the ZeroCash cryptographic flow.

2. Except for the Prover(=Holder) and Verifier, a third party, the *Issuer*,
must be here involved in order to record the Holder's commitment.
Note, however, that this involvement does not violate
requirement 4: it takes place *prior to* generation and transmission of
any proof of possession for $t$. More accurately, $I$ 's role is limited to
the preparation of context for these proofs, without ever involving to any
of them.

It should be finally stressed that the here proposed *APP* protocol is not
intended to replace *ZKD* and should not be considered as an improvement of it.
It is rather complementary to it in the context of a *Diplomata* application,
whenever the utmost degree of anonymity is desired. In fact, *APP* has some
practical drawbacks compared to *ZKD*:

1. Being due to non-interactiveness, anonymity of the Holder is at the cost
of the Verifier's ability to check for certificate revocation.
That is, if certificate revocation is supported by the system, then
the proposed proof of possession is only equivalent to a *proof of acquisition*.
This is because certification takes place once (e.g., upon issuance of the
title or whenever the Holder wishes)
and any subsequent revocation would need the Issuer's involvement
in order to be known. If the Verifier wants to verify that a title is
*still* possessed by the Holder, they would need to proceed to a second stage
and run some additional, possibly non-anonymous proof-verify session, where
the Issuer intermedieates in order to convey info about possible revocation.
2. Being a zk-SNARK protocol, *APP*'s setup presupposes a public ceremony for
destroying the randomness used in the generation of the
Common Reference String (CRS).

With these limitation in mind, we proceed to the exact description
of the protocol. Abstract aspects of the involved primitives
are exposed for reference in the Appendix.


## The Anonymous Proof of Possession (APP) Protocol

As usually, an append-only public ledger (say, blockchain) is available, where
all actions of network entities are recorded in the form of serial number.
Serial numbers can be randomly selected but unique for each
action; they are labeled with a tag, indicating the kind of action
to record in the ledger. Only a set of predefined tags is acceptable;
furthermore, special rules might determine which tags are allowed to be
used by the various network entities. For example, only issuers are allowed to
append serial numbers with the tag corresponding to title issuance.


### Preliminary discussion

Let $t$ be a title issued by $I$ to a holder $H$. The (issuance of the) title is
assigned by $I$ a unique serial number $s$, which $I$ possibly records in the
ledger. Note that this mere record conveys no info about either $t$
or $H$, beyond the fact that some title has been issued at some point of
history. Consequently, only $I$ can certify that $s$
relates to the particular title acquired by $H$.

This observation has major effect on the communication between $H$ and a
verifier $V$, to whom $H$ wants to prove possession of $t$: using $s$ alone in
order to verify possession of $t$ would necessarily need to involve $I$ and
communicate $s$ to them, violating the requirement of privacy. More importantly,
having assigned themselves $s$ to $t$,
$I$ can infer from $s$ both $H$'s identity and
$t$'s content. Clearly, in
order for identity and content to remain secret, $s$ can not be alone used
in the private communication between $H$ and $V$: if $H$ wants to
be able to prove possession without having $V$ appeal to $I$,
$s$ must be communicated to $V$ along with some additional quantity.

Due to the binding property, a natural choice is to use a commitment.
Assume that a statistically secure commitment mechanism $Comm$ is available.
Whenever $H$ wants to prove possession of $t$ to a verifier $V$, the holder
could, say, send a commitment $c=Comm(r, s)$ along with $s$, so that $V$
could later open the commitment with $r$ and verify that its content
is $s$. This scenario is severely flawed, since binding of
$H$ to $s$ does no way guarantee possession of $t$: $H$ can
commit to arbitrary serial numbers; even if $V$ audits the ledger to
check that $s$ appears in it, $H$ could still have committed to the serial
number of a title they don't possess. We thus need a further
binding of $s$ to $t$, i.e, a certification that $s$ is the serial number of
$t$. We know only $I$ can give this certification, so any
commitment used by $H$ must be beforehand publicly approved by $I$.

Assuming for the moment that such a certification mechanism is available, we
let $I$ approve and subsequently record $c$ in the ledger.
$H$ can now point to the public record $c$ and send $r$ to $V$, who
can then unlock $c$ with $r$ and verify its content.
However, consumption of $r$ by the verifier means that a new
commitment must be generated by $H$ for each proof-verify session
(otherwise a dishonest verifier could leak r to a malicious holder, who could
then successfully prove possession of $t$ by simply pointing to $c$ on the
public ledger). Consequently, $H$ must repeatedly appeal to $I$ for approval.
We can employ proof-of-knowledge for the purpose of generating only once a
commitment and thus minimize interaction with $I$. Instead of communicating
$r$ to $V$, the holder could equivalently prove knowledge of the particular
trapdoor $r$ which unlocks the $c$ public record. Indeed:

1. Since this record has been approved by $I$, verifying the proof
is equivalent to verifying possession of $t$ on behalf of $H$.

2. Since $r$ remains secret, $c$ may be uniformly reused by $H$ in all
proof-verify sessions concerning $t$.

What can this proof-of-knowledge mechanism be? In fact, the holder does not
even have to point to the $c$ public record: given $s$, and assuming that
approved commitments are recorded with a tag COMM indicating
their nature, $H$ needs only prove the following claim:

$$\text{"I know } r \text{ such that } Comm(r, s) \text{ appears in the public ledger with tag COMM" }$$

This statement is obviously NP-complete. Consequently, it can be produced as
a zk-SNARK proof upon the predicate

$$F(s, r) = Comm(r, s) \text{ appears in the public ledger with tag COMM }$$

The desired proof-of-knowledge mechanism can thus be a zk-SNARK
cryptosystem with public predicate $F$. The trapdoor $r$ serves
as the secret witness to the serial number $s$, which serves in turn as the
public instance of the predicate.

Till now we have only adjusted part of the *ZeroCash* protocol to the
*Diplomata* context, where awarded titles are interpreted as 1 BTC.
We haven't though specified how approval of commitments should proceed.
In ZeroCash, a holder's commitment to a particular bitcoin collection is
approved and recorded by means of an escrow (e.g., when physical money is
exchanged for electronic cash). In our protocol, this is equivalent to the
certification on behalf of $I$ that $c = Comm(r, s)$ is a commitment
to $s$: in order to record $c$ in the public ledger, the issuer must first
ensure that the holder has indeed committed to $s$ and not any arbitrary
value. On the other hand, the trapdoor $r$ must remain eternally secret.
In particular, $H$ cannot communicate $r$ to $I$ in order for the latter to
unlock the commitment and certify its content: a dishonest $I$ could leak
r to a malicious holder, who would then successfully prove possession of $t$ by
simply generating a proof on top of the zk-SNARK infrastructure. We
have to solve the following problem: how can the issuer certify
commitment to the correct serial number without knowing the secret trapdoor?

To do so, we will take advantage of the commitment's internal mechanism. The
scheme particularly suited for our purpose is the *Pedersen commitment scheme*,
as will be explained in the following section.    


#### Certification

Let $p$ be a large prime, $g$ a generator of the group of quadratic residues
in $\mathbb{Z}_p^\star$ and $h$ a randomly chosen residue. Fixing these public
parameters, the *Pedersen commitment scheme* is defined as

$$Comm(r, s) = g^r h^s$$

(involved operations taken $mod$ $p$). Security of the scheme (the hiding and
binding properties) depends on further assumptions upon $p$ and $g$ to be
stipulated when needed. We are here interested in the following key observation.

__*Lemma 1*__. Fix $p$, $g$, $h$ such that $Comm$ be statistically
secure. Let $c = Comm(r, s)$ be $A$'s commitment to a
secret $s$, which is known to both $A$ and $B$. If $A$ sends $g^r$
and $c$ to $B$, then

1. $B$ can check whether $c$ is a commitment to $s$ without knowing the
trapdoor $r$
2. $B$ cannot infer the value of $r$

Indeed, knowing already $s$, B can compute $h^s$ and check whether
$c$ is equal to $g^r h^s$. If $A$ had committed to any $s^\prime \neq s$,
i.e., $c = g^r h^{s'}$, then, in order to deceive $B$, they would need to
fabricate $r^\prime$ such that $g^r h^{s'} = g^{r'} h^s$ or, equivalently,
solve

$$r^\prime = \log g^r h^{s^\prime-s}$$

which is computationally infeasible in view of
discrete logarithm hardness. Based on the same hardness assumption, $B$
cannot infer $r$ from either $g^r$ or $h^s$ or their combinations.

The above observation applies to the *Diplomata* context by fixing the
Pedersen schema as the system's commitment mechanism. Let
a holder $H$ and an issuer $I$ have the roles of $A$ resp. $B$, $s$ be the
serial number of a title $t$ issued by $I$ to $H$ and $c$ the holder's
commitment to that number. In order to repeatedly use $c$ as certificate
in future proof-verify sessions, the holder requests from $I$ to approve
their commitment and append it to the public ledger. More accurately, after
committing with a randomly chosen trapdoor $r$, $H$ sends $s$, $g^r$
and $c$ to $I$ with the request to certify $c$ as commitment to $s$ and
subsequently append it to the ledger. $I$ first looks in their database of
awarded titles to see if $s$ corresponds indeed to any of them. If yes, $I$
proceeds to certification of $c$ by means of $g^r$. In case of success,
$I$ appends $c$ to the ledger; otherwise the holder's request is rejected.
The trapdoor remains unrevealed to the issuer as should.   


### Formal description

#### Setup

Let *L* denote the public ledger. Among possibly others, we define a tag

<p style="text-align: center;">COMM,</p>

indicating a commitment to an awarded title's serial number, and specify that
*only Issuers have the right to append records with this tag to the ledger*.

We fix a sufficiently high security parameter $n$ and randomly choose
$p = 2q + 1$ to be a strong (i.e., such that $q$ is prime)
$(n + 1)$-bit prime .
Denoting by $Q\subset\mathbb{Z}_p^\star$ the group of quadratic residues,
we randomly fix a generator $g$ of $Q$ and a residue $h \in Q$. We denote by

$$\mathbb{Z}_p^\star\times\mathbb{Z}_p\ni(r, s) \mapsto Comm(r, s) = g^r h^s$$

the Pedersen function over the pair $(g, h)$. Under the above
assumptions, discrete log hardness becomes plausible and $Comm$
may be considered as a statistically secure commitment mechanism.
We specify that *serial numbers assigned by Issuers to titles
be contained in* $\mathbb{Z}_p$.

We next consider the predicate

$$F(s, r) = Comm(r, s) \text{ appears in } L \text{ with tag COMM }$$

Both lookup in $L$ and evaluation of $Comm$ are of polynomial time complexity.
Consequently, for any $s$, the statement

$$\text{I know } r \text{ such that } F(s, r) = 1$$

is NP-complete with respect to $r$. This allows to use $F$ as the public
predicate of a zk-SNARK cryptosystem, where the trapdoor is thought of as
the private witness to the serial number. More accurately, feeding $F$ to a
CRS-generator, we denote by

<p style="text-align: center;">CRS = $(pk, vk)$</p>

the pair of produced proving and verifying keys of the induced zk-SNARK
cryptosystem. We will denote by

$$(\mathit{pk}, s, r) \mapsto \pi = \text{Prover}(\mathit{pk}, s, r),$$
$$(vk, s, \pi) \mapsto \text{Verifier}(\mathit{vk}, s, \pi) \in \{0, 1\}$$

the Prover, resp. Verifier algorithms of the zk-SNARK cryptosystem.
We specify that *every Holder can internally run a Prover
and Verifiers coincide with Verifiers of the* zk-SNARK *cryptosystem*.


#### Cryptographic flow

Let $t$ be a title issued by issuer $I$ to a holder $H$. Upon issuance of $t$,
the preliminary Steps 1-3 take place.

1. Issuer $I$ uniquely assigns to $t$ a serial number $s\in\mathbb{Z}_p$,
which they store in their local database as the identifier of $t$.
(For transparency or history reasons, $I$ might record $s$ in the public ledger,
but this is irrelevant to what follows.)

2. Issuer $I$ privately communicates $(s, t)$ to holder $H$.

3. Holder $H$ stores $s$ in their local database as the identifier of $t$.

In order for $H$ to be able to repeatedly prove possession of $t$,
they must enter at will the following certification phase (once per $t$).


##### Certification phase (once per title)

4. Holder $H$ detects in their local database the serial number $s$ of $t$.

5. Holder $H$ randomly chooses a trapdoor $r$ to compute $g^r$
and $c = Comm(r, s) = g^r h^s$.

6. Holder $H$ stores $r$ in their local database as the private witness
to $s$.

7. Holder $H$ privately communicates $(g^r, c, s)$ to $I$ with the
request to approve $c$ and append it to $L$.

8. Issuer $I$ looks through their local database to see if $s$ corresponds
indeed to a title awarded to $H$. If yes, issuer $I$ proceeds to Step 9;
otherwise $H$'s request is rejected and the certification phase terminates.

9. Issuer $I$ computes $h^s$ to check if $c = g^r h^s$. If yes,
$I$ proceeds to Step 10; otherwise $H$'s request is rejected and
the certification phase terminates.

10. Issuer $I$ appends $c$ to $L$ with tag COMM and informs $H$.

Note that a dishonest $I$ cannot avoid recording $c$ to the public ledger,
since a honest $H$ can anytime audit $L$ to see if their commitment has
been appended. Though not formally included in the protocol, it is a good
practice that $H$ audits $L$ to check if $c$ has been appended indeed.
On the other hand, nothing prevents a dishonest
$I$ from appending an incorrect commitment to the ledger. We will see that
this possibility does *not* affect the security of the protocol
(cf. the *Proof of security in the standard model* section).


##### Proof-verify session

Let a verifier $V$ meet anonymously $H$ online. $H$ wishes to prove that
they possess a title (in this case, $t$) by only referring to its serial number
$s$ (i.e., without revealing part of its content). Note that the relation
between $s$ and $t$ is only known to $I$ and $H$.

11. Holder $H$ detects in their local database the serial number $s$, which
identifies $t$ (cf. Step 3), and the private witness $r$ corresponding
to $s$ (cf. Step 6).

12. Holder $H$ runs the zk-SNARK Prover over $(r, s)$
to generate $\pi = \text{Prover}(\mathit{pk}, s, r)$

The produced $\pi$ is namely a zk-SNARK proof that $H$ knows $r$ such that
$Comm(r, s)$ has been publicly recorded by some certifying authority.
Equivalently, $\pi$ proves possession of an awarded title
on behalf of $H$.

13. Holder $H$ privately communicates $(\pi, s)$ to $V$.

14. Verifier $V$ feeds $(\pi, s)$ to the
zk-SNARK Verifier: $ans = \text{Verifier}(\mathit{vk}, s, \pi)$.

15. If $ans = 1$, verifier $V$ accepts $H$'s claim of knowledge;
otherwise the claim is rejected.


### Proof of security and honesty assumptions

We first justify security of the protocol from the purely cryptographic
viewpoint. No honesty assumptions are required with this respect.
Some honesty assumptions upon holders and issuers are necessary to
guarantee that secret parameters
are not leaked by the entities knowing them:
leakage of these parameters could potentially violate exclusiveness
of rights upon an awarded title or compromise anonymity of the holder.
Verifiers need not be honest.

In what follows, involved symbols refer to the above formal description.

#### Proof of security in the standard model

Standard references to stochastic tolerance are omitted and the discrete
log assumption is taken for granted.

*Completeness*: if $c$ is correct, i.e., $c = g^r h^s$, then $\pi$ verifies.

Assuming completeness of the certification phase, this follows directly
from completeness of the underling zk-SNARK cryptosystem. Consequently,
we need only check that $c$ is appended to $L$.
But this follows directly from Step 9 and the
auditability of the public ledger
(cf. the comment at the end of the *Certification phase* section).

*Soundness*: if $c$ is incorrect, i.e., $c = g^r h^{s'}$ with $s^\prime \neq s$,
then $\pi$ does not verify.

Due to soundness of the zk-SNARK cryptosystem, we need only consider
the case where $c$ has been approved by $I$ and appended to the ledger.
Substituting $H$ and $I$ for
$A$, resp. $B$ in Lemma 1.1, a honest $I$ would be
able to detect incorrectness of $c$ and thus abort the certification phase
(Step 8). Consequently, $I$ is dishonest in that they certify the incorrect
commitment $c$ on behalf of a dishonest holder $H$. However, in spite of
$g^r h^{s'}$ appearing in the ledger, $H$ cannot prove possession of $t$.
In view of Steps 13-15, $H$ would still need to
generate a verifiable zk-SNARK proof of the statement

<p style="text-align: center;">I know $r^\prime$ such that $g^{r'} h^s$ appears in $L$ with tag COMM,</p>

where $g^{r'} h^s = g^r h^{s'}$. More accurately, $H$ would need to compute

$$\text{Prover}(pk, s, r^\prime)$$

with $r^\prime = \log g^r h^{s^\prime - s}$,
contradicting the discrete log assumption.


#### Security considerations

Holder $H$ is normally the only entity with knowledge of $r$. Suppose
that $(r, s)$ is somehow communicated to another holder $H^\prime$ to be
a valid witness-instance pair. Then, by virtue of Steps 12-15 and the
completely public character of the zk-SNARK cryptosystem,
$H^\prime$ is able to prove for themselves possession of the title
identified by $s$. It thus makes sense to consider the following honesty
assumption:

<p style="text-align: center;">AS1. For the sake of their exclusive rights upon $t$,
holder $H$ never communicates $r$ to anybody.</p>

Raising this assumption pertains to the following scenario:
two dishonest holders $H$, $H^\prime$ take advantage of anonymity
and collaborate, so that the latter proves possession of a title awarded
to the former.

There is still another way to violate exclusivity of rights upon $t$,
stemming from a dishonest $I$ in collaboration with a dishonest holder
$H^\prime$. Suppose that $I$ communicates $s$ to $H^\prime$, who then
enters a certification phase with $I$: $H^\prime$ generates a correct commitment to
$s$, which $I$ maliciously (cf. Step 8) approves and appends to the ledger.
It is then possible for $H^\prime$ to prove for themselves possession of
the title awarded to $H$. It thus makes sense to consider the following honesty
assumption:

<p style="text-align: center;">AS2. For the sake of $H$'s exclusive rights upon $t$,
issuer $I$ never approves anybody else's commitment to $s$.</p>

#### Anonymity considerations

Issuer $I$ and holder $H$ are normally the only entities with knowledge that
$s$ is the serial number of $t$ (cf. Steps 1-3). Since $t$ contains most
probably reference to $H$'s identity, leakage of this relation could
compromise the latter's anonymity. For example, if $V$ had for some
reason accidental access to the content of $t$, then, after completion of
Step 13, $V$ would be able to infer $H$'s identity. It thus makes sense to
consider the following honesty assumption:

<p style="text-align: center;">AS3. For the sake of $H$'s anonymity,
neither $I$ nor $H$ ever communicates $(s, t)$ to anybody.</p>


### Performance optimization

#### Optimizing commitment detection for scaling

Detection of a particular record with tag COMM in the ledger
is crucial for the protocol and an operation to be massively repeated
(once for each proof-verify session), since it is involved in the
evaluation of the zk-SNARK public predicate. More accurately,
ledger lookup is incorporated in the CRS, which yields the fixed ingredient
$vk$ in the verification of any zk-SNARK proof. A naive
implementation of commitment detection can thus severely limit scalability,
since the verification algorithm most probably grows linearly with the
length of the public ledger.

We can remedy this by organizing certified commitments to an additional
linear structure, representing the leaves of a Merkle-tree $T$:
each time a commitment is certified,
$T$ is updated with the hash of this commitment.
In other words, tree $T$ records a specialized history of
certified commitments, so that one can directly audit $T$ instead of $L$
to see if a commitment has been approved by an issuer. It is well known
that update and leaf lookup for a Merkle-tree are of logarithmic time
and space complexity with respect to the number of leaves. Consequently,
auditing $T$ instead of $L$ reduces the complexity of $F$'s evaluation
from linear to logarithmic with respect to the number of records.

In integrating the Merkle-tree machinery with our
protocol, we proceed as follows. Upon setup, we
initialize an empty Merkle-tree $T$ with hash function $\tilde{h}$
and assume that a $\text{verify_merkle_proof}()$
function is available for verifying Merkle-proofs. Let
$T.\text{audit_proof}()$ denote $T$'s functionality for generating audit
proofs upon given records. In particular, a record $a$ has been
hashed and stored among $T$'s leaves if and only if

$$\text{verify_merkle_proof}(T.\text{audit_proof}(\tilde{h}(a))) = 1$$

With this notation, the zk-SNARK public predicate reformulates as

$$F(s, r) = \text{verify_merkle_proof}(T.\text{audit_proof}(\tilde{h}(Comm(r, s))))$$

In order for this to work, all we have to do is modify Step 10 of the
certification phase as follows:

<p style="text-align: center;">Issuer $I$ updates $T$ with $c$ and informs $H$</p>

Holder $H$ can then directly audit $T$ to check if their commitment has
indeed been certified. Note that, in this modified protocol,
usage of the COMM tag becomes inessential with respect to our specific
purposes. However, issuer $I$ may still record certified commitments in $L$
with tag COMM for history and transparency reasons.   


#### Accelerating verification

This is a generic zk-SNARK technique exclusively affecting the Setup
and Verifier algorithms of the underlying zk-SNARK cryptosystem.
See *Accelerating verification with a preprocessed key* in Appendix B.

## Application architecture

## Appendix

### A Generic commitment scheme

A (*statistically*) *secure commitment scheme* consists of a pair of functions

$$r, m \mapsto c = Comm(r, m)\text{,}$$

$m$ coming from a set of acceptable messages $M$ and $c$ referred to as
*commitment* to $m$ with *trapdoor* $r$, and

$$r, c \mapsto m = Open(r, c)$$

referred to as the *opening function*, such that

1. (*Correctness*) opening the commitment with the correct trapdoor yields the
original message, i.e.

$$Open(r, Comm(r, m)) = m$$

2. (*Hiding*) it is computationally infeasible to infer the original message
from the commitment alone, i.e., given $c = Comm(r, m)$, it is (with
overwhelming probability) impossible to find
$r'$ such that $m = Open(r^{\prime}, c)$

3. (*Binding*) it is computationally infeasible to anyhow extract from the
commitment a message other than the original, i.e., given $c = Comm(r, m)$,
it is (with overwhelming probability) impossible to find $r^{\prime}$ such that
$m^{\prime} = Open(r^{\prime}, c) \in M$ and $m^{\prime} \neq m$.

Let $A$ commit to a message $m$ with trapdoor $r$ and send the commitment $c$
to $B$. In view of correctness, the *hiding* property means that $c$ resembles
a box containing $m$ in the hands of $B$, whose key $r$ is controlled by $A$:
$B$ will never learn $m$ unless $A$ chooses to reveal $r$ to them. On the
counterpoint, again in view of correctness, the *binding* property means that
the box can be unlocked by $A$ in only one meaningful way: $A$
cannot fabricate a trapdoor so that opening the box with it leads
to any acceptable content other than the original.


### B Generic zk-SNARK scheme

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


#### Predicate reduction

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


#### Cryptographic flow

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

It is essential that $\mathit{lambda}$ remains eternally secret: leakage of
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

In this context, the zk-SNARK properties are formalized as follows:

* *zk* (*Zero Knowledge*): The generated proof conveys no info about
$w$ beyond the claim of it being known, i.e., that $F(x, .)$ is
satisfied by it.

* *S* (*Succinct*): The size of proofs is small, constant and
independent of the system's constraints and public inputs. More
accurately, omitting dependence on security parameters, proof
generation is $O(1)$ and verification is $O(k)$, where $k$
stands for the proof uniform bitsize. The bottleneck of initial
setup needs not be taken into account.

* *N* (*Non-interactive*): The only interaction between Prover and
Verifier is the transmission of the produced proof (one
communication step). Any intermediate computation required for proof
generation and verification has been absorbed in the precomputation
of the CRS (involved via usage of $\mathit{pk}$ and $\mathit{vk}$).

Generic requirements may also be formalized as follows:

* *Completeness*: Correct proofs, that is, generated upon a correct
witness, always (i.e., with overwhelming probability) verify.
More accurately, omitting reference to stochastic tolerance,

$$
\begin{align*}
&\textrm{for any } \pi =
\mathrm{Prover}(pk, x, w) \textrm{ such that } F(x, w) = 1,\\
&\textrm{there holds } \mathrm{Verifier}(vk, x, \pi) = 1
\end{align*}
$$

* *Soundness*: No false proof, that is, generated upon an incorrect
witness, ever (i.e., with overwhelming probability) verifies.
More accurately, omitting reference to stochastic tolerance,

$$
\begin{align*}
&\textrm{for any } x \textrm{ such that } F(x, .)
\textrm{ not satisfiable, there exists no } w \text{ such that } \\
&\mathrm{Verifier}(vk, x, \pi) = 1 \text{ with } \pi =
\mathrm{Prover}(pk, x, w)
\end{align*}
$$


#### Accelerating verification with a preprocessed key

For the purpose of faster verification, the verifying key $vk$
may be further processed during initial setup of the zk-SNARK cryptosystem,
yielding a so called *preprocessed* verification key $pvk$. In particular,
a small amount of extra precomputed info can be added to it, in which case the
Verifier algorithm must also be modified appropriately ("online"
Verifier).

Note that proof generation is not affected by the usage of
preprocessed verification keys.
