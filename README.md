kEvent an Arduino Event Manager
=====================

Based on the excellent Arduino Event Manager by Chris Brand.
https://github.com/arcturial/arduino-event


This library brings the principle of Events/Hooks to Arduino. In short, it allows a user
to respond to certain "events" quite easily.

### Table of Contents
* Installation
* Usage
* Timed Events
* Event Arguments
* Limitations


1. Installation
---------------

Clone or download the files in the repo and copy them into your Arduino library folder.

Include the library into your project with:

``` cpp

#include <kEvent.h>

```


2. Usage
--------

First off, we need to initialize a new EventManager. The EventManager will be responsible for registering new subscribers and triggering events when they occur.

``` cpp

EventManager evtManager;

```

Now lets define an event.

``` cpp
const EventID LED_ON(1);
```

Now that we have a new EventManager. Let's start listening for a new event.

``` cpp

EventManager evtManager;

const EventID LED_ON(1);

void setup() 
{
	evtManager.subscribe(Subscriber(LED_ON, &LEDListener));
}

```

Notice that we use a reference to a listener as our "callback". So we would need to create this callback. The callback struct needs to extend the EventTask interface.

``` cpp

EventManager evtManager;
int ledPin = 12;

const EventID LED_ON(1);

struct LEDListener : public EventTask
{
  using EventTask::execute;
  
  void execute(Event evt)
  {
  	digitalWrite(ledPin, HIGH);
  }

} LEDListener;

void setup() 
{
	pinMode(ledPin, OUTPUT);
	evtManager.subscribe(Subscriber(LED_ON, &LEDListener));
}

```

Now that we have a new listener registered for the event LED_ON, we actually need to send a signal to trigger the event.

``` cpp

EventManager evtManager;
int ledPin = 12;

const EventID LED_ON(1);

struct LEDListener : public EventTask
{
  using EventTask::execute;
  
  void execute(Event evt)
  {
  	digitalWrite(ledPin, HIGH);
  }

} LEDListener;

void setup() 
{
	pinMode(ledPin, OUTPUT);
	evtManager.subscribe(Subscriber(LED_ON, &LEDListener));
}

void loop()
{
	evtManager.trigger(LED_ON);
}

```

There we go, an event to light the LED at pin 12 triggers every "loop" interval. This might seem a bit redundant, but it's just an example. Event's won't be of much use for small applications with one purpose, they do however make large applications with many commands easier to maintain.


3. Timed Events
---------------

You can set up intervals to trigger certain events every X amount of milliseconds.

``` cpp

const EventID KEEP_ALIVE(2);

void setup()
{
  evtManager.triggerInterval(TimedTask(2000, KEEP_ALIVE));  
}

void loop()
{
  evtManager.tick();
}

```

This piece of code will set up the EventManager to trigger a new "event.keepAlive" event every 2000 milliseconds (2 seconds). You must call the tick() function, so that the event manager knows where in time it is.

4. Event Arguments
------------------

You can also use one event to do multiple things. Let's say we want one LED event to either turn it on or off.

``` cpp

...

const EventID LED_STATE(4);

struct LEDListener : public EventTask
{
  using EventTask::execute;
  
  void execute(Event evt)
  {
  	if (evt.extra == 1)
  	{
  		digitalWrite(ledPin, HIGH);
  	}
  	else
  	{
  		digitalWrite(ledPin, LOW);
  	}
  }

} LEDListener;

...

void loop()
{
	evtManager.trigger(LED_STATE, 1);
	delay(1000);

	evtManager.trigger(LED_STATE, 0);
	delay(1000);
}

```

This will turn raise a new "on" event and one second after that, raise an event to turn the LED off again. Thus, this will result in a "blinking LED" sketch.

5. Limitations
--------------

This library was mainly written for programming with the Arduino UNO. Since it has limited memory, there is a hardcoded limit on the amount of events.

``` cpp

#define INTERVAL_SLOT_COUNT 1  // 1 available interval slot
#define SUBSCRIBER_SLOT_COUNT 3  // 3 available subscriber slots

```

This can be changed by defining the values your self just before including the header file for the first time.
