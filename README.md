# EventSpeakerTrigger


Develop a Trigger on Event - Speaker object which will throw an error if
the Speaker Selected on Event - Speaker Record already have an Event against his name. i.e - For a Speaker there will be only one event at a time. 

Reject Duplicate Bookings

 and throw the error: 'The speaker is already booked at that time’.

Q1 - In which object the Trigger will be ( Event - Speaker )
Q2 - What are the events ( before insert , before update )

```java
trigger EventSpeakerTrigger on EventSpeakers__c (before insert, before update
```

// Step 1 - Get the speaker id & event id
// Step 2 - SOQL on Event to get the Start Date and Put them into a Map
// Step 3 - SOQL on Event - Spekaer to get the Related Speaker along with the Event Start Date
// Step 4 - Check the Conditions and throw the Error

//Step 1

```java
Set<Id> speakerIdsSet = new Set<Id>();
    Set<Id> eventIdsSet = new Set<Id>();
    
    for( EventSpeakers__c es : Trigger.New ){
         speakerIdsSet.add(es.Speaker__c);
         eventIdsSet.add(es.Event__c);
    }
```

// Step 2

```java
Map<Id, DateTime> requestedEvents = new Map<Id, DateTime>();
    
    List<Event__c> relatedEventList = [Select Id, Start_DateTime__c From Event__c 
                                       Where Id IN : eventIdsSet];
    
    for(Event__c evt : relatedEventList ){
        requestedEvents.put(evt.Id, evt.Start_DateTime__c);
    }
```

// Step 3

```java
List<EventSpeakers__c> relatedEventSpeakerList = [ SELECT Id, Event__c, 
                              Speaker__c, Event__r.Start_DateTime__c
                                         FROM EventSpeakers__c
                                         WHERE Speaker__c 
                                         IN : speakerIdsSet];
```

// Step 4
for( EventSpeakers__c es : Trigger.New ){ 
        
       DateTime bookingTime = requestedEvents.get(es.Event__c); 
// DateTime for that event which is associated with this new Event-Speaker Record
        
        for(EventSpeakers__c es1 : relatedEventSpeakerList) {
            if(es1.Speaker__c == es.Speaker__c && es1.Event__r.Start_DateTime__c == bookingTime ){
          es.Speaker__c.addError('The speaker is already booked at that time');
                es.addError('The speaker is already booked at that time');
            }
        }
        
    }
