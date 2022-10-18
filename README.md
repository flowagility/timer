# FlowAgility Timer Integration (v.1.0.0)

The following document describing the message formats and protocal for message exchange between the timer and FlowAgility platform. If you want your timer to be supported by FlowAgility and want to test it or having any questions or doubts, please contact us at info@flowagility.com

## Contents
- [General concept](#general-concept)
- [Connecting Timer and Platform](#connecting-timer-and-platform)
- [Keeping Connection Alive](#keeping-connection-alive)
- [11(+1) Symbols Message Format](#111-symbols-message-format)
- [Platform to Timer Communication](#platform-to-timer-communication)
- [Timer to Platform Communication](#timer-to-platform-communication)

## General concept
FlowAgility platform is supporting timer integration in an experimental mode at the moment. Communication is supported based mainly on 11 symbols codes with few exceptions. Timer is connecting to the platform automatically, sending its unique Mac Address. Platform is openning a communication channel for a specific Ring and Date. Further commincation is done via web socket. Platform is sending an information on Faults, Refusals and Eliminations, while timer is sending the message when dog is crossing an entrance gates and also the exit gates. Platform can also initiate a Course Walk countdown. Platform can reset timer at any moment.

## Connecting Timer and Platform
To connect the timer to a platform, the platform should be set into a listening mode, by providing the Mac Address of a timer. This can be done inside of the run views. After the timer Mac Address is entered, the platform is waiting for the timer to connect. Timer should then initiate connection to a specific URL (https://flowagility.com/ws/timer/mac_address - where mac_address is a Mac Address of the timer). Timer should try to connect to the platform with series of 5, 10, 30 and 60 seconds intervals (3-5 attempts in a series).

As soon as timer is connected, the platform is ready to receive signals from the timer.

Eveytime the user is loading the page which is able to track timer connection, the platform is sending a status request to the timer and is awaiting for the response with a current state. 

Pages, which are monitoring the timer connection, are usually tracking the following events:
- Status report
- Timer start
- Timer stop
- Course walk start
- Course walk stop
- Timer connection dropped

## Keeping Connection Alive
Timer is responcible for keeping connection alive, using an appropriate `ping` command.

## 11(+1) Symbols Message Format
Reguilar message from the timer consists of 11 symbols (1 alpha-code and 10 digits).
Alpha codes are the following:
  - `g` - start of a Course Walk with a given time
  - `i` - timer running with F+R+E information
  - `o` - stop of a Course Walk
  - `p` - timer stopped with F+R+E information
 
Digital codes are the following
  - 1 digit - Numebr of Faults (0..9)
  - 2 digit - Number of Refusals (0..9)
  - 3 digit - No elimination is `0`. Eliminated is `1`
  - 7 digits: elapsed millisecond (so max elapsed will be 9999 secs)

Example:
```text
i2100036597
  ││││└─┬─┘
  ││││  └─────> 36597 milliseconds
  │││└────────> not eliminated
  ││└─────────> 1 refusal
  │└──────────> 2 faults
  └───────────> timer real time info message		
```

Response from the Timer is always pre-pended by a `#`symbol. Example above in case of the response, should look like this `#i2100036597`.
When platform is sending information about faults, refusals of eliminations, timer should respond the same F+R+E and also current time in milliseconds.


## Platform to Timer Communication
The following commands are send by the platform:
  - `d0` - status request. Response should be pre-pended by `#` symbol.
  - `i1000000000` - reporting 1st fault while timer is running (timer should be ignored by the timer)
  - `i2000000000` - reporting 2nd fault while timer is running
  - `i0100000000` - reporting 1st refusal while timer is running
  - `i0200000000` - reporting 2nd refusal while timer is running
  - `i0010000000` - reporting elimination (pure elimination or elimination by Refusals or Elimination by MCT)
  - `o000420000` - setting or pausing the Course Walk time (default is 7 minutes)
  - `g000420000` - starting ot resuming the Course Walk time 
  - `p000000000` - timer reset
  - `A.B.C` - version response of the platform, where A is the major, B minor, C revision of the current version of the cupported API.
  - `pong` - response to `ping` command from Timer

## Timer to Platform Communication 
The following commands are send by the timer:
  - `#..........` - responce with the current timer state should be pre-pended by the `#` symbol and should follow 11 symbols format
  - `i0000000000` - timer start
  - `#i2100036597` - response with infromation shown on the running timer at the moment of the request
  - `p2100036597` - timer stoped with 2 faults, 1 refusals, not eliminated and with 36.597 seconds
  - `version` - request for an API version
  - `ping` - ping message to a platform, that should receive `pong` as a response
  
