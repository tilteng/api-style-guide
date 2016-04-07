# Atari - The Tilt Payments System

Atari is Tilts payment router and gateway system.

The goal of Atari is to have it handle *logical*
movements of funds in the Tilt Ecosystem.  It currently also handles the
*Physical* movment of funds, though that responsibility may eventually be
extracted into it's own library or service.

Atari is responsible for managing account
balances, escrows, tracking fees, and ensuring that Tilt doesn't pay out
more than was brought in for any particular escrow/account.


# Developing on Atari

Currently, development on Atari is strictly working towards isolation of Atari
from other components/modules/packages in the Tilt API.  In order to achieve
this, the following rules should be followed on any code changes in the
`Atari::` namespace:


* Don't add (and ideally remove) `Tilt::*`, `Crowdtilt::*`, and `Dancer::*`
   Imports, Packages, and Objects from the `Atari::` Namespace
* Atari should not accept ANY new objects outside of the `Atari::` namespace (no
    `Tilt::*`, `Crowdtilt::*`, `Dancer::*`, etc.)
* During the refactoring, any outside objects brought into the `Atari::`
    namespace, should be treated as immutable, and never modified within Atari

