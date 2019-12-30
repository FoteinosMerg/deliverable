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
the claim that it exists among their acquired qualifications
3. in one single communication step
4. without involvement of anybody else in that step
(say, the Issuer of $t$).

The proposed protocol will be referred to as
*Anonymous Proof of Possession Protocol* (*APP*). Proof of possession should be
thought of as proof of *inclusion* in a set of recordedly awarded titles
(possibly of a special kind) strongly correlated to the prover's (unrevealed)
identity. In doing so, we employ the *zk-SNARK* cryptographic
scheme combined with a commitment mechanism. In particular, proof of possession
is a zk-SNARK proof.

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
requirement 3). More specifically, $I$ is restricted to once participating in
the common setup of these proofs, without actually involving to any of them.

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

## The Anonymous Proof of Possession (APP) Protocol

### Preliminaries

We expose here the cryptographic flow we want to achieve in view of the
requirements 1-4.

#### Commitment mechanism


### Protocol description

#### Setup

#### Cryptographic flow

Let $t$ be a title held by a Holder $H$ and issued by Issuer $I$. In order for
$H$ to be able to anonymously prove possession of $t$ at any future moment, the
following Steps 1-3 must take place (once per t).

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

*I* needs never again interact with either *H* or any possible verifier. Their
role is limited in approving the record *c*, which serves as disguise of *s*,
and thus *t*, in the public ledger. This disguise guarantees that proofs of
possession of *t* are indeed "zero-knowledge" with respect to it, since they
will only contain references to *c*.

#### Anonymization via public predicate

### Performance and storage optimization

#### Optimizing proof generation

#### Optimizing verification

## Appendix

### A Generic zk-SNARK scheme
### B Merkle-proof of inclusion
