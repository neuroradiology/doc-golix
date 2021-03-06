# Current revision: 29 March 2016

+ Changed dynamic bindings to use monotonic counter for replay protection and frame indexing, instead of a hash chain of frame GHIDs.
+ Changed dynamic bindings to use a target vector instead of current target and history separated
+ Updated dynamic binding version to 16
+ Updated spec and security whitepaper to reflect the change (however, GOBD diagramming still needs revision)

# Previous revisions

## 29 March 2016, #edfd65e

+ Clarified use of dynamic binding historical frames
+ Removed support for multiple targets in dynamic bindings; incremented version accordingly.
+ Updated debindings to be stateless; incremented version accordingly.
+ Explained debinding chains
+ Added salt definition in key agreement
+ Clarified requested GHID and author/recipient in asymmetric requests.
+ Clarified valid targets for static, dynamic bindings
+ Changed "pipe request" to "pipe handshake" to differentiate from "asymmetric request"
+ Potential deprecation warning in ciphersuite 2
+ Removed optional ghid request in persistence provider pings, and clarified several responses to/from persistence providers.
+ Added "query debinding" persistence provider command
+ Modified behavior of "list bindings" persistence provider command to return the binding ghids instead of their authors
+ List bindings (the persistence provider command) now has a specified order.

## 28 Feb 2016, #8b925e8

+ Rebranded Muse -> Golix. No version changes as a result, though. Corrected some minor glaring errors in the process, and noted some things in the spec as deprecated (but did not redocument them yet).
+ Added text explicitly forbidding dynamic binding to anything other than a static GEOC
+ Fixed overlooked update in GDXX record re: chain count vs chain length. Updated version accordingly.
+ Changed pipe request identifier to ASCII "RQ"

## 27 Feb 2016, #6aff066

+ Updated dynamic binding to remove support for multiple identical start points, and removed old wording surrounding random seed for that purpose. Upped version number accordingly.

## 26 Feb 2016, #4bf3e65

+ Reorganized spec documentation to list cipher suites and address algos first
+ Added dedicated identity container (GIDC)
+ Added ancillary format for secret sharing

## 20 Feb 2016, #e27e7bf

+ Major readme revision for better expressibility
+ Removed shameless plugs
+ Switched numeric identification of ciphersuites 1 and 2 in preparation for possible removal of SIV mode
+ Changed lengths of any GHID lists into absolute lengths, to ease support of potential future address algorithms of different lengths. Updated version numbers accordingly.
+ Moved all of the asymmetric operations into a single primitive (GARQ) with differing payloads. Added another payload definition for arbitrary content, to support future protocol development.
+ Clarified available size of asymmetric payload and appropriateness of use
+ Changed all secondary file extensions to .ghid and tertiary to object identifiers (ex .geoc)
+ Added support for different static/dynamic address algorithms in dynamic binding

## 18 Jan 2016, #4fcba8f

+ Clarified spec re: salt length in signature (should match hash length = 64 bytes)
+ Added some shameless plugs

## 11 Dec 2015, #9a6e2ff

+ Changed all DH from 2048-bit/Group #14 to ECDH/Curve25519
+ Changed external links to reflect muterra.io and ethyr.net reorganization

## 13 Nov 2015, #10d6a60

+ Removed deterministic nonce generation in spec
+ Changed cipher suite 0x1 to SIV mode with MAC prepended to ciphertext in spec
+ Added cipher suite 0x2 as CTR mode with nonce accompanying key distribution in spec
+ Added important questions to top of readme

## 10 Nov 2015, #83eb7e5

Initial draft release.