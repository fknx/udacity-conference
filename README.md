# udacity-conference

### Setup Instructions

The setup instructions can be found in the **Readme.md** file in the **ConferenceCentral** folder.

### Task 1

#### Explain your design choices

The sessions are created as conference descendants since this way the 1 to n relationship between conferences and sessions can be stored easily and session don't make a lot of sense without a conference. For simplicity, the duration is stored as minutes in an `IntegerProperty`. 

The highlights are stored as repeated `StringProperty`. I interpreted the highlights to work like tags, so that you can specify some key points of the session (for example "C#" and "ASP.NET").

The speaker (his name ;)) is stored simply in a `StringProperty`. I deemed that the name is the only important property of the speaker so creating an extra entity seemed unnecessary effort. However, in a real life application I would probably use a speaker entity, as most likely you would want to store additional information (such as contact information) and not just the name.

I extended the profile to store the user's wishlist. I added a repeated `KeyProperty` that stores the keys of the sessions on the wishlist. This way the profiles and sessions are only loosely coupled and there will be no issues in case a profile or session is removed.

### Task 3

#### Additional queries

##### Get sessions by date

Can be used to retrieve all sessions on a certain date.
	
	try:
	    date = datetime.strptime(request.date, "%Y-%m-%d").date()
	except Exception, e:
	    raise endpoints.BadRequestException('The date must be in the format year-month-day.')
	
	# fetch all sessions of the given speaker
	sessions = Session.query().filter(Session.date == date)
	

#### Get sessions by highlights

Returns all sessions with at least one of the specified highlights. Useful in case the user has a list of interests and wants to see all sessions that cover one of these interests.

	if not request.highlights:
        raise endpoints.BadRequestException('You have to query for at least one highlight.')

    # fetch all sessions with at least one of the given highlights
    sessions = Session.query(Session.highlights.IN(request.highlights))

#### Query related problem

The problem is that this query requires two inequality conditions on two different properties, which is not supported by ndb. One way to solve this problem is to let ndb only filter by one of the conditions and then programmatically check the other condition to remove all non-matching entries from the result set. For example, one could fetch all non-workshop session using ndb and then only return those starting before 7 pm.

The following code was used to implement the query for the `getAllNonWorkshopSessionsBefore7PM` endpoint:

	# fetch all sessions which are not a workshop
    sessions = Session.query().filter(Session.typeOfSession != 'workshop')

    sevenPm = time(19, 00)

    # convert the sessions to session form objects and return them
    return SessionForms(
        items=[self._copySessionToForm(session) for session in sessions if session.startTime < sevenPm]
    )