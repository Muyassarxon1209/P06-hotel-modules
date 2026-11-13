# P03 design notes

## Why design before code
Low-level design decides which classes exist, what information each class keeps and which methods it has, and which class calls which, before we type the code. Drawing it first means a rule such as "does this stay share a night?" is written once (in DateRange.Overlaps) and every screen uses that one copy.

## Where the methods live
- **Cancel()** lives in **Reservation**, because cancelling changes the reservation's own status, and only the reservation knows and owns that status. (Nothing is deleted, so the history is kept.)
- **HasOverlap(room, stay)** lives in **ReservationStore**, because only the store can see all the reservations and so can answer "does anyone already have this room on these nights?".

## Activity traces
**Omar, R7, 13–15 November (Maya has 12–14).** First diamond: is check-out after check-in? Yes. Second diamond: does a Confirmed booking share a night? Yes, the 13th. The request ends at "Unavailable · save nothing".

**Nora, R7, 14–16 November.** First diamond: is check-out after check-in? Yes. Second diamond: does a Confirmed booking share a night? No (Maya leaves on the 14th, so there is no shared night). The request continues: save as Confirmed, send the confirmation email, and ends at "Accepted + reservation id" (R102).

**Omar, R8, 15–15 November.** First diamond: is check-out after check-in? No (15 is not after 15). The request ends at "InvalidDates · save nothing". The second diamond is never reached.

## Prediction
I expect Nora's line to be **Accepted R102**: her stay 14–16 does not share a night with Maya's 12–14, and Maya's R101 is already saved, so the next id is R102.

## The race
Prediction: R7 will have **2** Confirmed reservations for 12–14 November.

What happened: Maya's and Omar's requests both asked "is R7 free?" before either one saved, so both got "false" (no overlap) and both saved, giving R101 and R102 for the same room and nights. The problem appears between the two calls in `Reserve`: between `HasOverlap(...)` (the check) and `Save(...)` (the act), where another request can change the answer. This is a race condition, and requirement R3 is broken. P09 will fix it in the database.


# P04 design notes

## Refactoring
In a refactoring the structure of the code changes (names, files, what depends on what), but what the app does does not change.

Must stay the same:
- The tests from Step 2: all 6 still pass after every move.
- The demo output: the same nine lines as in P03.
- The race demo: still "Double booking!", because refactoring does not fix bugs.

## What our code breaks
- BookingService asks for exactly ReservationStore (D: dependency inversion).
  In P07 we would have to edit BookingService just to use a database.
- BookingService asks for exactly EmailSender (D and O: dependency inversion and open/closed).
  Adding SMS confirmations would mean editing BookingService's constructor and Reserve(), and in a test we could not make the email fail to prove that the booking is kept.

## Proof
- Tests: 6 passed (python -m unittest -v), before adding R6 and SMS.
- Demo: the same nine lines as in P03.
- Race: still double-booked ("Double booking!"), as expected; P09 fixes it.


# P06 design notes

## Before
- Tests: 8 passed (python -m unittest -v).
- Demo: the nine lines, the same as in P03.
- Race: still double-booked ("Double booking!").

## Modules

| Module | Public contract (others may use) | Internal code (only the module, the wiring and the tests) |
|---|---|---|
| Rooms | Room, the room catalog promise | InMemoryRoomCatalog |
| Reservations | BookingService, BookingResult, Reservation, DateRange, Guest, ReservationStatus, the store promise | InMemoryReservationStore |
| Payments | PaymentStatus, the payment service promise | SimulatedPaymentService |
| Notifications | Confirmation, the notification sender promise | EmailNotificationSender, SmsNotificationSender |

## Architecture rules
1. Reservations uses only the public contracts of other modules: BookingService may use the Notifications contract, never the email sender.
2. Rooms does not depend on any other module: it knows nothing about reservations, money or messages.
3. Payments and Notifications do not depend on Reservations: the arrows point one way, from Reservations outwards.

## Broken on purpose
Rule 1 failed ("Reservations uses only the public contracts of other modules"): the message named hotel_booking.reservations.booking importing hotel_booking.notifications.internal. The tests stayed green, so only the architecture check caught it. I removed the line and all checks were green again.

## After
- Tests and checks: 8 tests passed and 3 import-linter contracts kept (11 green).
- Demo: the same nine lines as before.
- Race: still double-booked, as expected; P09 fixes it.
