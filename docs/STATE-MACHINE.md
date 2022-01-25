# State Machine

MindBindJS uses a finite state machine (FSM) to control complexity and make verification of correctness easier.

The app has the following flow for a Pomodoro:

* The user presses start on the Pomodoro screen, which starts a timer.
  * The timer begins to run down, and will do so even while the app is backgrounded. If the app is closed, however, it is discarded.
* The user may pause the timer, and then resume it (TODO: decide if this should be allowed).
* The user may notify the app that they are in a *meeting*, or more generally are in a block of time that is potentially unbounded.
  * This allows the user to incorporate a Pomodoro-based workflow with things that don't fit into a single Pomodoro.
  * It also helps notify the app that the user isn't slacking off when they otherwise would be treated as such.
* When the timer expires, the app emits a bell sound and the app transitions to the 'Feedback' page.
* In the Feedback stage, a timer is started and the user decides on a score from 0-10 (inclusive) indicating how productive they felt the session to be and optionally a short written summary.
  * The pair of a numeric score and potentially empty text message are wrapped as a single Feedback object and stored in a History.
* If the user has not completed their feedback in the allotted time, the rest of the time is considered 'dead time' and not productive.
* When the user submits their feedback, the feedback is saved and the app transitions to the 'Breakroom' page.
* The Breakroom stage automatically starts a timer that will return the user to the Pomodoro page when it expires.

In terms of states, the following is a reasonable characterisation:

* Pomodoro-start-initial (first Pomodoro when app is opened)
* Pomodoro-start-delayed (after first Pomodoro, returning to the Pomodoro page)
* Pomodoro-timer-resumed
* Pomodoro-timer-paused
* Pomodoro-timer-meeting
* Feedback-start
* Feedback-delayed
* Breakroom

In terms of legal transitions (without specifying exactly the conditions for transitioning):

01. Pomodoro-start-initial -> Pomodoro-timer-resumed
02. Pomodoro-timer-resumed -> Pomodoro-timer-paused
03. Pomodoro-timer-paused -> Pomodoro-timer-resumed
04. Pomodoro-timer-paused -> Pomodoro-timer-meeting
05. Pomodoro-timer-resumed -> Pomodoro-timer-meeting
06. Pomodoro-timer-meeting -> Pomodoro-timer-resumed
07. Pomodoro-timer-meeting -> Pomodoro-timer-paused
08. Pomodoro-timer-resumed -> Feedback-start
09. Pomodoro-timer-meeting -> Feedback-start
10. Feedback-start -> Feedback-delayed
11. Feedback-start -> Breakroom
12. Feedback-delayed -> Breakroom
13. Breakroom -> Pomodoro-start-delayed
14. Pomodoro-start-delayed -> Pomodoro-timer-resumed

This is a sensible implementation, although some transitions could be added in the future. What about the conditions/events for these transitions?

01. User presses start button
02. User presses pause button
03. User presses start button
04. User presses meeting button
05. User presses meeting button
06. User presses meeting button
07. User presses pause button
08. Pomodoro timer expires
09. User presses meeting button + Pomodoro timer expires
10. Feedback timer expires
11. User submits feedback
12. User submits feedback
13. Breakroom timer expires
14. User presses start button

Seems pretty simple actually - could have slightly smarter logic eventually, but this looks good!
