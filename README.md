App Engine application for the Udacity training course.

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.

## Design Choices

### Task 1

Because sessions are always part of a conference, they are created with the conference entity as their parent. This mean means that a session, it's parent conference, and the conference's parent user all form an entity group. This allows us to make strongly consistent queries to the datastore that negates the possibility of stale data within the entity group.

Speakers may have been implemented as entities as well. The only data being tracked for a speaker is a name however, so I decided to just use a string field on each session for the speaker, it is an easy and definitely simpler approach. If more data needed to be stored for each speaker, it would then become worthwhile to set them up as a separate entity.

### Task 2

User wishlisted sessions are implemented by storing the session keys as repeated properties of each user entity. When a user adds a session to their wishlist, it's key is added to that user's profile entity session keys on wishlist property.

### Task 3

#### Additional Queries

1. Implemented a query to return only those sessions that have a particular highlight in their list of highlights. The end point path is "getSessionsWithHighlight".

1. Implemented a query to return only those session for a particular date and that have a specified maximum duration. The end point path is "getSessionsOnDateWithMaxDuration".

#### Query Problem

"Solve the following query related problem: Let’s say that you don't like workshops and you don't like sessions after 7 pm. How would you handle a query for all non-workshop sessions before 7 pm? What is the problem for implementing this query? What ways to solve it did you think of?"

The difficulty in solving this query in the NDB Datastore is caused by the limitation of not being able to use more than one inequality filter in a query. 

Ordinarily I would apply these 2 filters to the query:

```python
        # query sessions for those that DO NOT match the type of session
        sessions = Session.query()
        # order session query by inequality filter (type of session)
        sessions = sessions.order(Session.typeOfSession)
        filteredSessions = sessions.filter(Session.typeOfSession !=
                                           request.typeOfSession)
        filteredSessions2 = filteredSessions.filter(Session.startTime <=
        											request.startTime)
```
Because of the current NDB query limitation of only 1 inequality filter I used the following process.

Use 1 inequality NDB filter (the type of session), and then manually filter out the other criteria (start time) by iterating over the returned filtered query and only adding the records that pass this 2nd criteria to a new array to return:

```python
        # query sessions for those that DO NOT match the type of session
        sessions = Session.query()
        # order session query by inequality filter (type of session)
        sessions = sessions.order(Session.typeOfSession)
        filteredSessions = sessions.filter(Session.typeOfSession !=
                                           request.typeOfSession)

        # manually query for sessions that are earlier than the latest desired
        # start time
        filteredSessions2 = []
        for sess in filteredSessions:
            if sess.startTime not None and sess.startTime <= filterTime:
                filteredSessions2.append(sess)
```

### Task 4

Added feature to get the featured speaker of a conference. Each time a session is added to a conference, the sessions for that conference are checked to see which speaker now has the most sessions. The speaker with the most sessions is recorded in the featured speaker key within MEMCACHE and can then be retrieved via the "getFeaturedSpeaker" end point. Note, if 2 speakers have the same number of sessions in a conference, the last of those speakers added will be the new featured speaker.

[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool
