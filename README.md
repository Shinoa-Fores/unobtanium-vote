# unobtanium vote

## Description

This program implements a voting system based on data from the blockchain (addresses and balances).
Given a block number (the date of the vote), each address with a positive balance can vote, and the weight of the vote is the balance of the address.

It is written in Common Lisp.

## Dependencies

* A Common Lisp implementation. Tested with:
  * [sbcl](http://www.sbcl.org)
  * [clozurecl](http://ccl.clozure.com)
  * [ecl](http://ecls.sourceforge.net)
  * [cffi](http://www.cliki.net/CFFI)
  * [cl-base64](http://www.cliki.net/cl-base64)
  * [ironclad](http://cliki.net/Ironclad)
  * [split-sequence](http://www.cliki.net/SPLIT-SEQUENCE)
  * The crypto library from [OpenSSL](http://www.openssl.org)

## Installation

* Install [quicklisp](http://www.quicklisp.org/beta/) to manage the packages.
* Install the dependencies: ```(ql:quickload '("cffi" "cl-base64" "ironclad" "split-sequence"))```
* Copy the source code of the unobtanium blockchain parser where you want it to be.
* Tell your Common Lisp implementation where to find the sources:
  * ```(push "directory-where-the-sources-are/" asdf:*central-registry*)```
  * If you don't want to type this line every time, you can add it to the initialization file (e.g.: .sbclrc, .ccl-init.lisp, .eclrc).

## Usage

First, you must prepare the data required for the vote:

* A file describing the motion the vote is about.
* A file containing the hash of the motion (that is *sha256(sha256(motion file))*). Il can be generated with the *make-motion-hash* function in the *tools* directory.
* A file containing the candidates (depending on the motion, it could be yes, no, blank, names of people, etc.).
* A file containing the unobtanium addresses and their balances at the block number chosen as the time of vote. It can be generated with the *make-balance-file* function in the *tools* directory if you have the blockchain in a database created with the [cl-unobtanium](http://unobtanium.uno/devel/cl-unobtanium/) program.
* A file containing the votes.

Then, you can load the vote package and compute the results of the vote:

    (require 'unobtanium-vote)
    (unobtanium-vote:vote "motion-hash" "candidates.txt" "votes.txt" "balances.txt")

## File formats

### Candidate file

The candidate file must have one candidate per line.

Example:

    yes
    no
    blank

### Balance file

The balance file must have one "address balance" pair per line.
The balance must be in µUNO, in order to have only integers and avoid floating point number rounding issues.

Example (with testnet addresses):

### Vote file

The vote file must have one "address vote signature" tuple per line.
The format of the vote must be "motion-hash:candidate".
The signature must be the base64 encoded signature of the vote (just the "motion-hash:candidate" string without new line after it) by the private key associated with the address. It can be generated with the message signing feature of the unobtanium wallet.

Example (with testnet addresses):

A vote is considered as invalid if:

* the address doesn't exist.
* the balance of the address if 0.
* the address has already voted.
* the motion hash doesn't match.
* the candidate doesn't exist.
* the signature is incorrect.

