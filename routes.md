# Tilt API Routes

In the Tilt API, we are trying to move away from using Dancer's anonymous route
handlers (via `get`,`put`,`post`, etc.) as much as possible, and remove
Dancer as a dependency from the core business logic in our route code.

To help with this, we've introduced the `Tilt::Controller` class.  This class
allows us to write route code as object methods, which have the `RequestContext`
passed to them, rather than relying on `Dancer` and it's global state.  It also
enables us to decouple our routes from `Dancer`.

Examples of using `Tilt::Controller` can be found here:

* UserPayouts
    * https://github.com/tilteng/api/blob/dev/lib/Crowdtilt/Internal/API/User/Financials/Payouts.pm
    * https://github.com/tilteng/api/blob/dev/lib/Tilt/Controller/UserPayouts.pm
    
