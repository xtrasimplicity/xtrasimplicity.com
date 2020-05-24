---
title: "Google Calendar API - Fetching Group calendars"
date: 2020-05-24T18:53:27+10:00
draft: false
tags: [
  'Google',
  'API',
  'Calendar',
  'Groups',
  'Google API',
]
categories: []
---

I'm currently working on writing a sync application to migrate Group calendars from Google G Suite to MS O365, and after reading up on a lot of APIs etc, I thought I'd post a short summary of my findings below.

Unfortunately, Google's Calendar API isn't as flexible as I'd like it to be, so this is a much more complex process than it arguably should be.

The following are some caveats/"gotchas" that I've come across so far.

### API caveats
- Google's API doesn't allow you to impersonate a Group, even with domain-wide delegation.
  - As a result, you can't simply get a [CalendarList](https://developers.google.com/calendar/v3/reference/calendarList/list) for a Group email, as you would for a user.
- Google's API doesn't allow you to fetch calendar lists on an organisational-level.
  - So if you want to get all calendars in an organisation, you'll have to iterate through each user and build up an array of unique calendars manually.

### Group caveats
- Groups don't have calendars - Users do, and they're *shared with* a group.
  - Each [event](https://developers.google.com/calendar/v3/reference/events) has an `Organizer` attribute with the calendar's unique ID as the `id`, and the calendar's unique email address (i.e. `domain.tld_uniqueID@group.calendar.google.com`) as the `email`. There's no reference to the group's normal email (i.e. `groupname@domain.tld`). As expected, the `creator` attribute reflects the user who created the event.
- Each user who has a Group's calendar in their Calendar list retrieves the same Calendar ID. This can be used to cross-reference calendars between users.
  - e.g. with the Ruby client API, the `Google::Apis::CalendarV3::CalendarListEntry` object will have the same `etag` and `id` for different users.
- There does not appear to be a way to programmatically determine the associated group email address, from a CalendarList.


Based on this, to be able to get a list of unique shared calendars within an organisation, you'd need to:

 1. Create an array or Hash map to be used to track the shared calendars within an organisation. 
 2. Get a list of users.
 3. For each of the users
  - Get all calendar lists for this user,
  - Filter the calendar lists by those with `id`s ending with `@group.calendar.google.com`,
  - Add these into the aforementioned array or Hash map.
