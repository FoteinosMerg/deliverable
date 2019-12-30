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
*no honesty* and *auditability* by external observers are also satisfied.

A *ZK-Decryptor Protocol* (*ZKD*) has already been proposed on top of an
El-Gamal cryptosystem, that allows holders to (indirectly) prove possession of
titles to verifiers. However, the Issuer of the title is heavily involved in
the process. Except for an initial certification request and the private
communication of the accompanying commitment to the Verifier, the rest of
communication takes place between Issuer and Verifier: it is the *Issuer* who
actually commits and generates the proof of award that will be finally verified.
More importantly, verification of proof presupposes that the (content of the)
title will be decrypted by the Verifier and thus *disclosed* to them, i.e.,
"zero-knowledge" does not refer to the qualification itself. Since the title
contains as such reference to the Holder's identity, this entails
*non*-anonymity for the Holder.

Purpose of this text is to propose an alternative to the *ZKD* protocol
for the case where the Holder's anonymity is desired and the (content of the)
title should not be revealed. This pertains to the following meritocratic
scenario: Holder and Verifier meet anonymously online and the Holder wishes to
prove *for themselves* possession of a title $t$

1. without revealing their identity (public key)
2. without disclosing any info about $t$, except for
the fact that it exists among their acquired qualifications
3. in one single communication step
4. without involvement of anybody else in that step
(say, the Issuer of $t$).

The proposed protocol will be referred to as
*Anonymous Proof of Possession Protocol* (*APP*). Proof of possession should be
thought of as proof of *inclusion* in a set of recordedly awarded titles
(possibly of a special kind) strongly correlated to the prover's (unrevealed)
identity. In doing so, we employ the *zk-SNARK* cryptographic
scheme combined with a commitment mechanism. In particular, proof of possession
will be a zk-SNARK proof.

Such a construction will partly resemble the *ZeroCash*
anonymous payment scheme, with each title interpreted as 1 BTC (bitcoin) and
the holders' commitments recorded in the public ledger.
There are however two major differences:

1. No "Direct Anonymous Payment" method is here facilitated, that is, transfer
of titles(=coins) from holder to holder is *not* anyhow taken into account.
This allows to stay away from the intricacies of the key-address machinery
surrounding commitments in the ZeroCash cryptographic flow.

2. Except for the Prover(=Holder) and Verifier, a third party, the *Issuer*, is
responsible for recording the Holder's commitment: contrary to the commitment
to a coin's serial number, the Holder's commitment to a title's serial number is
appended to the ledger not by the Holder, but by the Issuer (see Step 3 below
for details). Note however that this involvement does *not* violate
requirement 4: it takes place *prior to* generation and transmission of
any proof of possession for $t$ (the communication step mentioned in
requirement 3). More specifically, $I$ 's role is limited to the context
preparation of these proofs, without even being involved to any of them.

It should be finally stressed that the here proposed *APP* protocol is not
intended to replace *ZKD* and should not be considered as an improvement of it.
It is rather complementary to it in the context of a *Diplomata* application,
whenever the utmost degree of anonymity is desired. In fact, *APP* has some
practical drawbacks compared to *ZKD*:

1. Anonymity is at the cost of the Verifier's inability to check for certificate
revocation. That is, if certificate revocation is supported by the system, then
the proposed proof of possession is only equivalent to a *proof of acquisition*.
In order for the Verifier to verify that a title is *still* possessed by a
Holder, they will have to proceed to a second stage and run some additional,
possibly non-anonymous proof-verify session (say, in the *ZKD* context).
2. Being a zk-SNARK protocol, *APP*'s setup presupposes a public ceremony for
destroying the randomness used in the Common Reference String (CRS) generation.

With these limitation in mind, we proceed to the exact description
of the protocol.

## The Anonymous Proof of Possession (APP) Protocol

### Setup

As usually, an append-only public ledger (say, blockchain) is available, where
all actions of network entities are stored in the form of serial numbers.
Serial numbers can be randomly selected but unique for each action; they are
accompanied by a label, indicating the kind of action to record in the ledger.
Only predefined labels are considered to be acceptable serial numbers;
furthermore, special rules might determine which labels are allowed to be
recorded by each kind of network entity.


#### The COMM label   

#### Commitment mechanism

#### CRS generation

### Cryptographic flow

Let $t$ be a title held by a Holder $H$ and issued by Issuer $I$. In order for
$H$ to be able to anonymously prove possession of $t$ at any future moment, the
following Steps 1-3 must take place (once per t).

#### Proof-context preparation

1. Upon issuance of $t$, $I$ uniquely assigns to it a serial number $s$, which
they record to the public ledger and privately communicate to $H$.

2. $H$ requests from $I$ to record their commitment to $s$ in the public ledger.
In particular, $H$ chooses a secret trapdoor $r$ and computes a commitment
$$c = Comm(r, s)$$
which they privately communicate to $I$. This commitment is generated
once per $t$ and will be used in any subsequent prove-verify session concerning
this title.

3. Being the issuer of $t$, $I$ verifies and approves $H$'s commitment to $s$,
which they subsequently append to the public ledger. Note that only the issuer
$I$ of $t$ should be able to approve $c$ and record it in the ledger. In the
cryptocurrency terminology, approval on behalf of $I$ is equivalent to a
guaratee that $H$ has spent 1 BTC to an escrow pool. $H$ can anytime
check whether $c$ has indeed been recorded, since the ledger is public.

#### Anonymization via public predicate

*I* needs never again interact with either *H* or any possible verifier. Their
role is limited to once approving the record *c*, which will eternally serve as
disguise of *s* (and thus *t*) in the public ledger. This disguise guarantees
that proofs of possession for *t* are indeed "zero-knowledge" with respect to
its content, since they will only contain references to *c* and *s* cannot be
inferred from the latter. More accurately, a proof of possession for $t$
amounts to proving the following NP-complete statement:

$$\text{"I know } r \text{ such that } Comm(r, s) \text{ appears in the public ledger with label COMM"}$$

or equivalently, using the fixed zk-SNARK public predicate,

$$\text{"I know } r \text{ such that } F(r, s) = 1\text{"}$$

#### Proof-verify session

### Performance and storage optimization

#### Optimizing proof generation

#### Optimizing verification

## Notes on the application architecture

## Appendix

### A Generic commitment scheme

A (*statistically*) *secure commitment scheme* consists of a pair of functions

$$r, m \mapsto c = Comm(r, m)\text{,}$$

where $c$ is referred to as *commitment* to the *message* $m$ with
*trapdoor* $r$, and

$$r, c \mapsto m = Open(r, c)$$

referred to as the *opening function*, such that

1. (*Correctness*) opening the commitment with the correct trapdoor yields the
original message, i.e.

$$Open(r, Comm(r, m)) = m$$

2. (*Hiding*) it is computationally infeasible to infer the original message
from the commitment alone, i.e., given only $c = Comm(r, m)$, it is (with
overwhelming probability) impossible to find $r'$ such that $m = Open(r^{\prime}, c)$

3. (*Binding*) it is computationally infeasible to extract from the commitment a
message other than the original, i.e., given $c = Comm(r, m)$, it is (with
overwhelming probability) impossible to find $r^{\prime}$ such that
$Open(r^{\prime}, c) \neq m$

The following application is of practical interest: let *A* commit to a
message $m$ with trapdoor $r$ and send the commitment $c$ to *B*. The
*hiding* property means that $c$ is like a box containing $m$ in the hands of
*B*, whose key $r$ is controlled by *A*: *B* will never learn $m$ until
*A* chooses to reveal *r* to them. *Correctness* means that the box $c$ indeed
contains $m$, so that *B* really learns $m$ if *A* chooses to reveal *r*.
The *binding* property means that the box $c$ can only contain $m$: *A* cannot
falsify a trapdoor so that opening the commitment leads to any meaningful
content except for the original.

### B Generic zk-SNARK scheme
### C Merkle-proof of inclusion
