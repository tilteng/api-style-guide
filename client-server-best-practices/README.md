# Business Logic on the Server, Presentation Logic on the Clients

Avoid the clients needing to know business logic like "is the user an organizer of this campaign?" in order to display a button.  Links are a good pattern to allow this.  See [https://github.com/tilteng/api/wiki/Links-%26-Feature-Discovery]Links & Feature Discovery] for more information.

At our best we are building APIs that allow clients to be as dumb and agnostic to the business as possible.  More logic on the server allows us to consolidate logic between all three of our clients, minimizes the surface area for bugs, and allows us to change client behavior without forcing uses to download the newest version of the app.

Example:

Users can invite friends to a campaign using `POST /campaigns/:id/invites`.  Users are going to invite friends through a button in the website and the mobile clients.

However, campaigns that are expired can't have invites.  We wouldn't want to show this button in the case that someone is viewing an expired campaign, so we'll want to hide it or otherwise gray it out.

One way to accomplish this would be for the server to return `is_expired` as a flag in the campaign object, and have the client optionally display the button based on whether or not `is_expired` is set.

However, this is a fragile design.  If there are other reasons why the client needs to forbid invites (for example, a campaign has had inviting explicitly disabled), every client must now update.  Additionally, since the server controls access to `POST /campaigns/:id/invites`, this logic must also be duplicated on the server.

A better solution is whether the server presents the capability to invite to the client.  One pattern to do this is by including the invite href as a "link" subobject on the campaign.  Here are some example links:

```
"campaign": {
	"links": {
		/* determine whether invite button should be shown */
		"invites.create": {
			"href": "/campaigns/CMPB546BAC556AF42DEB45AC83C31169F0B/invites",
			"method": "POST"
		},
		/* determine whether "remind all" button should be shown */
		"reminders.create": {
			"method": "POST",
			"href": "/campaigns/CMPB546BAC556AF42DEB45AC83C31169F0B/invite_reminders"
		}
	}
}
```

The "remind all" feature is organizer-only.  With this design, the client only needs to look for the presence of the "reminders.create" on the campaign object to know whether or not to add a button - which means that the client does not have to add logic about whether the logged in user is the organizer.  This design easily supports other access levels with a minimum of client changes - if a client is coded only to display the button only on the presence of the "reminders.create" link, a "co-organizer" who is authorized to remind can be easily supported.

# Single User Interaction -> Single Client Request

Single button press - single API call
Single view - constant number of API calls (avoid n+1 calls for table rows)

# Performance

Run DBIC_TRACE=1 DBIC_TRACE_PROFILE=console to understand what your routes are doing
Run explain on your queries
Avoid n+1 SQL calls
Optimize the columns you fetch (join vs prefetch)

# Error Messages

Human-Readable 

# Pagination

Use marker-based pagination whenever possible.  This avoids the clients needing to do logic to construct what request to make for the "next page"

Sorting - ideally controlled by server-side
	Invites: for organizer, remindable people at top, joined at bottom