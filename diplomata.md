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
Requirements like *access control* on behalf of the holders, resistance to
*no honesty* and *auditability* by external observers should also be satisfied.

A *ZK-Decryptor Protocol* (*ZKD*) has already been proposed on top of an
El-Gamal cryptosystem, that allows holders to (indirectly) prove possession of
titles to verifiers. However, the Issuer of the title is heavily involved in
the process. Except for an initial certification request and the private
communication of the accompanying commitment to the Verifier, the rest of
communication takes place between Issuer and Verifier: it is the *Issuer* who
actually commits and generates the proof of award that will be finally verified.
More importantly, verification of proof presupposes that the (content of the)
title will be decrypted by the Verifier and thus *disclosed* to them, i.e.,
"zero-knowledge" does not refer to the qualification' content. Since the title
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
thought of as proof of *inclusion* in a set of recordedly awarded titles
(possibly of a special kind) strongly correlated to the prover's (unrevealed)
identity. In doing so, we will employ the *zk-SNARK* cryptographic
scheme combined with a commitment mechanism. In particular, proof of possession
will be a zk-SNARK proof.

Such a construction will partly resemble the *ZeroCash*
anonymous payment scheme, with each title interpreted as 1 BTC (bitcoin) and
the holders' commitments recorded in the ledger.
There are, however, two major differences:

1. No "Direct Anonymous Payment" method is here facilitated, that is, transfer
of titles(=coins) from holder to holder is *not* anyhow taken into account.
This allows to stay away from the intricacies of the key-address machinery
surrounding commitments in the ZeroCash cryptographic flow.

2. Except for the Prover(=Holder) and Verifier, a third party, the *Issuer*, is
responsible for recording the Holder's commitment: contrary to the commitment
to a coin's serial number, the Holder's commitment to a title's serial number is
appended to the ledger not by the Holder, but by the Issuer.
Note, however, that this involvement does *not* violate
requirement 4: it takes place *prior to* generation and transmission of
any proof of possession for $t$. More specifically, $I$ 's role is limited to
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
This is because approval of certification takes place upon issuance of the title
by the Issuer and a subsequent revocation would need the latter's involvement
in order to be known. If the Verifier wants to verify that a title is
*still* possessed by the Holder, they would need to proceed to a second stage
and run some additional, possibly non-anonymous proof-verify session, where
the Issuer intermedieates in order to convey info about possible revocation.
2. Being a zk-SNARK protocol, *APP*'s setup presupposes a public ceremony for
destroying the randomness used in the Common Reference String (CRS) generation.

With these limitation in mind, we proceed to the exact description
of the protocol.

## The Anonymous Proof of Possession (APP) Protocol

As usually, an append-only public ledger (say, blockchain) is available, where
all actions of network entities are recorded in the form of serial number.
Serial numbers can be randomly selected but unique for each
action; they are accompanied by a label, indicating the kind of action
to record in the ledger. Only a set of predefined labels is acceptable;
furthermore, special rules might determine which labels are allowed to be
used by the various network entities. For example, only issuers are allowed to
append serial numbers with the label corresponding to title issuance.

### Preliminary discussion

Let $t$ be a title issued by $I$ to a holder $H$. The action of issuance is
assigned by $I$ a unique serial number $s$, which $I$ possibly records in the
ledger. Note that this mere record conveys no info about either $t$
or $H$, beyond the fact that some title has been issued at some point of
history. Consequently, only $I$ can certify that $s$
relates to the particular title acquired by $H$.

This observation has major effect on the communication between $H$ and a
verifier $V$, to whom $H$ wants to prove knowledge of $t$: using $s$ alone in
order to verify possession of $t$ would necessarily need to involve $I$ and
communicate $s$ to them, violating the requirement of privacy. More importantly,
having assigned $s$ to
the fact of issuance, $I$ can infer from $s$ both $H$'s identity and
$t$'s content. Clearly, in
order for identity and content to remain secret, $s$ can not be alone used
in the private communication between $H$ and $V$. If $H$ wants to
be able to prove possession without having $V$ appeal to $I$,
$s$ must be communicated to $V$ along with some additional quantity.

Due to the binding property, a natural choice is to use a commitment.
Assume that a statistically-secure commitment mechanism $Comm$ is available.
Whenever $H$ wants to prove possession of $t$ to a verifier $V$, the holder
could, say, send a commitment $c=Comm(r, s)$ along with $s$, so that $V$
could later open the commitment with $r$ and verify that its content
is $s$. This scenario is severely flawed, since binding of
$H$ to $s$ does no way guarantee possession of $t$: $H$ can
commit to arbitrary serial numbers; even if $V$ audits the ledger to
check that s appears in it, $H$ could still have committed to the serial
number of a title they don't possess. We thus need a further
binding of $s$ to $t$, i.e, a certification that the serial number of $t$'s
issuance is $s$. We know only $I$ can give this certification, so any
commitment used by $H$ must be beforehand publicly approved by $I$.

We let $I$ approve and subsequently record $c$ in the ledger.
$H$ can now point to the public record $c$ and send $r$ to $V$, who
can then unlock $c$ with $r$ and verify its content.
However, consumption of $r$ by the verifier means that a new
commitment must be generated by $H$ for each proof-verify session
(otherwise a dishonest verifier could leak r to a malicious holder, who could
then successfully prove possession of $t$ by simply pointing to $c$ on the
public ledger). Consequently, $H$ must repeatedly appeal to $I$ for approval.

We can employ proof-of-knowledge for the purpose of generating only once a
commitment and thus minimizing interaction with $I$. Instead of communicating
$r$ to $V$, the holder could equivalently prove knowledge of the particular
trapdoor $r$ which unlocks the $c$ public record. Indeed:

1. Since this record has been approved by $I$, verifying the proof
is equivalent to verifying possession of $t$ on behalf of $H$.

2. Since $r$ remains secret, $c$ may be uniformly reused by $H$ in all
proof-verify sessions concerning $t$.

What can this proof-of-knowledge mechanism be? In fact, the holder does not
even have to point to the $c$ public record: given $s$, and assuming that
approved commitments are recorded with a label COMM indicating
their nature, $H$ needs only prove the following claim:

$$\text{"I know } r \text{ such that } Comm(r, s) \text{ appears in the public ledger with label COMM" }$$

This statement is obviously NP-complete. Consequently, it can be produced as
a zk-SNARK proof upon the predicate

$$F(r, s) = Comm(r, s) \text{ appears in the public ledger with label COMM }$$

In other words, the desired proof-of-knowledge mechanism can be a zk-SNARK
cryptosystem with public predicate $F$. The trapdoor $r$ serves
as the secret witness to the serial number $s$, which serves in turn as the
public instance of the predicate.

Till now we have only adjusted part of the *ZeroCash* protocol to the
*Diplomata* context, where awarded titles are interpreted as 1 BTC.
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

#### Certification mechanism

Let $p$ be a large prime, $g$ a generator of the group of quadratic residues
in $\mathbb{Z}_p^\star$ and $h$ a randomly chosen residue. Fixing these public
parameters, the *Pedersen commitment scheme* is defined as

$$Comm(r, s) = g^r h^s$$

(involved operations taken $mod$ $p$). Security of the scheme (the hiding and
binding properties) depends on further assumptions upon $p$ and $g$ to be
stipulated when needed. We are here interested in the following key observation:
Let $c = Comm(r, s)$ be the Pedersen commitment of *A* to a secret $s$, which
is known to both *A* and *B*. If *A* sends $g^r$ and $c$ to *B*, then

1. *B* can check whether $c$ is a commitment to $s$ without knowing the
trapdoor $r$
2. *B* cannot infer the value of *r*

Indeed, knowing already $s$, B can compute $h^s$ and check whether
$c$ is equal to $g^r h^s$. If *A* had committed to any $s^\prime \neq s$,
i.e., $c = g^r h^{s'}$, then, in order to deceive *B*, *A* would need to
fabricate $r^\prime$ such that $g^r h^{s'} = g^{r'} h^s$ or, equivalently,
solve

$$r^\prime = \log g^r h^{s^\prime-s}$$

which is computationally infeasible in view of
discrete logarithm hardness. Based on the same hardness assumption, *B*
cannot infer *r* from either $g^r$ or $h^s$ or their combinations.

The above observation applies in the *Diplomata* context by fixing the
Pedersen schema as the system's commitment mechanism. Let
a holder $H$ and an issuer $I$ have the roles of *A* resp. *B*, $s$ be the
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


### Cryptographic flow

#### Setup

<!-- ...

Let $t$ be a title held by a Holder $H$ and issued by Issuer $I$. In order for
$H$ to be able to anonymously prove possession of $t$ at any
future moment, the following Steps 1-4 must take place (once per t) -->


#### Certification phase (once per title)

<!-- 1. Upon issuance of $t$, $I$ uniquely assigns to it a serial number $s$. (For
transparency or history reasons, $I$ might record $s$ in the public ledger,
but this is irrelevant to what follows.)

2. $I$ privately communicates $s$ to $H$.

3. $H$ requests from $I$ to record their commitment to $s$ in the public ledger.
In particular, $H$ chooses a secret trapdoor $r$ and computes a commitment
$$c = Comm(r, s)$$
which they privately communicate to $I$. This commitment is generated
once per $t$ and will be used in any subsequent prove-verify session concerning
this title.

4. Being the issuer of $t$, $I$ verifies and approves $H$'s commitment to $s$,
which they subsequently append to the public ledger. Note that only the issuer
$I$ of $t$ should be able to approve $c$ and record it in the ledger. In the
cryptocurrency terminology, approval on behalf of $I$ is equivalent to a
guaratee that $H$ has spent 1 BTC to an escrow pool. $H$ can anytime
check whether $c$ has indeed been recorded, since the ledger is public. -->

#### Proof-verify session

### Performance and storage optimization

#### Optimizing proof generation

#### Optimizing verification

## Notes on the application architecture

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

Let *A* commit to a message $m$ with trapdoor $r$ and send the commitment $c$
to *B*. In view of correctness, the *hiding* property means that $c$ resembles
a box containing $m$ in the hands of *B*, whose key $r$ is controlled by *A*:
*B* will never learn $m$ unless *A* chooses to reveal *r* to them. On the
counterpoint, again in view of correctness, the *binding* property means that
the box can be unlocked by *A* in only one meaningful way: *A*
cannot fabricate a trapdoor so that opening the box with it leads
to any acceptable content other than the original.

<!-- The following scenario is of practical interest: *B* knows a secret *m* and
wants to verify that *A* knows $m$ too, but *A* is unwilling to communicate *m*
explicitly. Instead, *A* hides the content by sending a commitment $c$ to *B*,
which *B* can use at any later moment to verify knowledge of $m$ on behalf of
*A*. More specifically, whenever *B* wishes to verify *A*'s claim of knowledge,
he ask her to send him the trapdoor $r$ used at commitment phase. *B* uses $r$
to open the commitment: if the result is $m$, *B* accepts the claim; otherwise,
due to binding, the claim is rejected. -->

### B Generic zk-SNARK scheme
### C Merkle-proof of inclusion
