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

A protocol (*ZK-Decryptor Protocol*) has already been proposed on top of an
El-Gamal cryptosystem, that allows holders to (indirectly) prove possession of
titles to verifiers. However, the Issuer of the title is heavily involved in
the process. Except for an initial certification request and the private
communication of the corresponding commitment to the Verifier, the rest of
communication takes place between Issuer and Verifier: it is the *Issuer* who
generates the proof of award that will be finally verified. More importantly,
verification of proof presupposes that the (content of the) title will finally
be *disclosed* to the Verifier, i.e., "zero-knowledge" does not refer to the
qualification itself. Since the title contains as such reference to the Holder's
identity, this entails *non*-anonymity for the Holder.

Purpose of this text is to propose an alternative to the *ZK-Decryptor* protocol
for the case where the Holder's anonymity is desired and the (content of the)
title should not be revealed. This pertains to the following meritocratic
scenario: Holder and Verifier meet anonymously online and the Holder wishes to
prove *for themselves* possession of a title

1. without revealing their identity (public key)
2. without disclosing any info about the title, except for
the claim that it exists among their acquired qualifications
3. in one single communication step
4. without involvement of anybody else in that step
(say, the Issuer of the title)

The proposed protocol will be referred to as
*Anonymous Proof of Possession Protocol*. Proof of possession should be
thought of as proof of *inclusion* in a set of recordedly awarded titles
(possibly of a special kind) correlated to the prover's (unrevealed) public key
-without further reference. In doing so, we employ the *zk-SNARK* cryptographic
scheme combined with a commitment mechanism on top of the already available
El-Gamal cryptosystem. Such a construction will partly resemble the *ZeroCash*
anonymous payment scheme, with each title interpreted as 1 BTC
(bitcoin) and the holders' commitments to serial numbers being recorded in the
public ledger. There are however two major differences:

1. No "Direct Anonymous Payment" method is here facilitated, that is, transfer
of titles(=bitcoins) from holder to holder is *not* anyhow taken into account.
This allows to stay away from the intricacies of the key-address machinery
surrounding commitments in the ZeroCash cryptographic flow.

2. Except for the Prover(=Holder) and Verifier, a third party, the *Issuer*, is
involved in the generation of the Holder's commitment: contrary to a bitcoin's
serial number, a title's serial number, to which the Holder commits, is not
chosen by the Holder themselves but given to them by the Issuer of the title.
Note however that this involvement does *not* violate requirement 4.: it takes
place *prior to* the generation and transmission of the zk-SNARK proof of
possession (the single communication step mentioned in 3.), i.e.,
the Issuer is restricted to only determining the public instance of that proof.

It should be finally stressed that the here proposed protocol does not intend
to replace the *ZK-Decryptor Protocol*, being rather complementary to it in the
context of a *Diplomata* application. In fact, the *ZK-Decryptor* setup is
a partial prerequisite of our protocol, since the latter will need to utilize
the Holder's private key in generating commitments on top of an El-Gamal
cryptosystem.

## The Anonymous Proof of Possession Protocol

### Preliminaries

We expose here the cryptographic flow we want to achieve in view of the
requirements 1-4.

#### Commitment mechanism

#### Anonymization and the public predicate

### Protocol description

#### Setup

#### Cryptographic flow

### Performance and storage optimization

#### Optimizing proof generation

#### Optimizing verification

## Appendix

### Schnorr protocol
### Generic zk-SNARK scheme
### Merkle-proof of inclusion
