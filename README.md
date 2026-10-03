# EventForce Management System

A Salesforce CRM application for an event planning company. It keeps clients, events,
venues, vendors and feedback in one org and automates the repetitive parts of booking
an event.

Built on a Salesforce Developer Edition org (Lightning Experience).

**Team:** Yogeswari S (leader), Vaishnavi G, Vijaysree A, Pabhisri V, Sruthi M
**College:** Arasu Engineering College

## Features

- Six custom objects: Event, Client, Vendor, Venue, Feedback, Event Vendor (junction)
- Email validation rule on Client
- Event Budget formula field based on Event Type
- Approval process for event cancellation, with email alerts
- Record-triggered Flow that emails the client 3 days before the event
- Apex trigger that blocks two events at the same venue on the same date
- Apex trigger and helper that set a venue to Reserved (event Confirmed) or Available (event Canceled)
- Scheduled Batch Apex job that marks past events as Completed (batch size 200)
- Lightning App "Event planner", report "Upcoming Events by Month" and an operations dashboard

## Apex code in this repository

| Component | Type | What it does |
|---|---|---|
| `EventTrigger13` | Trigger (after insert, after update on Event__c) | Calls `VenueStatusHelper` |
| `VenueStatusHelper` | Class | Updates `Venue__c.Availability_Status__c` based on `Event_Status__c` |
| `PreventDoubleBooking` | Trigger (before insert, before update on Event__c) | Rejects a save when the venue is already booked on that date |
| `ScheduleCompleteEvents` | Schedulable class | Starts `BatchCompleteEvents` with a scope of 200 |
| `BatchCompleteEvents` | Batch class | Finds events dated before today that are not Completed and sets them to Completed |

Clicks-based configuration (validation rule, formula, approval process, Flow) is
described in [`docs/declarative-config.md`](docs/declarative-config.md).

## Project structure

```
force-app/main/default/
  classes/     VenueStatusHelper, ScheduleCompleteEvents, BatchCompleteEvents
  triggers/    EventTrigger13, PreventDoubleBooking
docs/          declarative-config.md
sfdx-project.json
```

## How to deploy

The custom objects and fields must exist in the target org first (Event__c, Venue__c,
Client__c and their fields), because the Apex code refers to them.

1. Install the [Salesforce CLI](https://developer.salesforce.com/tools/salesforcecli).
2. Log in to your org: `sf org login web`
3. Deploy the code: `sf project deploy start --source-dir force-app`
4. Schedule the nightly job from the Developer Console (Anonymous Apex), for example:

```apex
System.schedule('Complete Past Events Nightly', '0 0 20 * * ?', new ScheduleCompleteEvents());
```

(The cron expression above runs every day at 8:00 PM.)

## Future scope

- Apex test classes (code coverage is currently 0%; 75% is required for production)
- Online payment integration
- Client portal or mobile app
- More reports and dashboard charts
 ## Project Documentation
[View the documentation (PDF)](EventForce_Documentation_Yogeswari.pdf)

## Demo Video
[Watch the demo video here](https://youtu.be/UnPdUxf0Q9I)
