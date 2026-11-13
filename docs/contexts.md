# Hotel contexts

## Words

| Word | Meaning here | Name in our code |
|---|---|---|
| Stay | Check-in to check-out, half-open: 12–14 Nov = the nights of the 12th and 13th | DateRange |
| Reservation | Guest + room + stay + status, with an id such as R101 | Reservation |
| Confirmed | The room is promised to this guest for these nights | ReservationStatus.CONFIRMED |
| Guest | A person who stays at the hotel, with a name and an email (Maya, maya@example.com) | Guest |
| Room | A room the hotel sells, with a number such as R7 | Room |
| Room type | Standard or Deluxe; it decides the nightly price | Room.type |
| Nightly price | What one night in a room costs; Deluxe costs more than Standard | not in code yet |
| Out of service | A room taken off sale, for example R8 for repairs next week | not in code yet |
| Cancelled | The promise is withdrawn; the nights are free again | ReservationStatus.CANCELLED |
| Book | Ask for a free room for a stay; the answer is Accepted, Unavailable or InvalidDates | BookingService.reserve(guest, room, stay) |
| Cancel | Withdraw a reservation; it stays saved as Cancelled | BookingService.cancel(reservation_id) |
| Free rooms | Rooms with no Confirmed reservation that shares a night with the stay | BookingService.find_free_rooms(stay) |
| Deposit | 30% of the stay's price, taken when the guest books | not in code yet |
| Paid | The deposit has been received | not in code yet |
| Declined | The payment was refused; no deposit is taken | not in code yet |
| Confirmation message | The message that tells the guest the reservation is confirmed | NotificationSender.send_confirmation(reservation) |
| Sent | The message was handed to the email or SMS provider | not in code yet |
| Failed | The provider could not send the message; the reservation is not touched | not in code yet |

## Rooms

Job: knows which rooms the hotel sells.

| Word | Meaning here |
|---|---|
| Room | A room the hotel sells, with a number such as R7 |
| Room type | Standard or Deluxe; it decides the nightly price |
| Nightly price | What one night in a room costs; Deluxe costs more than Standard |
| Out of service | A room taken off sale, for example R8 for repairs next week |

Rules this context owns:
- A room that is out of service is not offered to guests.
- The nightly price depends on the room type (Deluxe costs more than Standard).

Needs from other contexts: nothing.
Tells other contexts: nothing by itself; it answers Reservations' question "which rooms exist?".

## Reservations

Job: decides whether a guest gets a room for some nights, and keeps every reservation.

| Word | Meaning here |
|---|---|
| Stay | Check-in to check-out, half-open: 12–14 Nov = the nights of the 12th and 13th |
| Reservation | Guest + room number + stay + status, with an id such as R101 |
| Confirmed | The room is promised to this guest for these nights |
| Cancelled | The promise is withdrawn; the nights are free again |
| Guest | A person who stays at the hotel, with a name and an email |
| Book | Ask for a free room for a stay; the answer is Accepted, Unavailable or InvalidDates |
| Cancel | Withdraw a reservation; it stays saved as Cancelled |
| Free rooms | Rooms with no Confirmed reservation that shares a night with the stay |

Rules this context owns:
- A room never has two Confirmed reservations that share a night.
- Check-out must be after check-in.
- Cancelling a reservation frees its nights for new bookings.

Needs from other contexts: the list of rooms (Rooms); from P17 also a deposit (Payments).
Tells other contexts: ReservationConfirmed (Notifications).

## Payments

Job: takes deposits and records whether they were paid.

| Word | Meaning here |
|---|---|
| Deposit | 30% of the stay's price, taken when the guest books |
| Paid | The deposit has been received |
| Declined | The payment was refused; no deposit is taken |

Rules this context owns:
- The deposit is 30% of the stay's price.
- Only a Paid deposit can be refunded.

Needs from other contexts: the reservation id and the amount to take (sent by Reservations; the nightly price behind the amount comes from Rooms).
Tells other contexts: the answer Paid or Declined (to Reservations).

## Notifications

Job: tells guests what happened, by email or SMS.

| Word | Meaning here |
|---|---|
| Confirmation message | The message that tells the guest the reservation is confirmed |
| Sent | The message was handed to the email or SMS provider |
| Failed | The provider could not send the message; the reservation is not touched |

Rules this context owns:
- A failed message is recorded as Failed and never changes the reservation.

Needs from other contexts: the ReservationConfirmed event (from Reservations).
Tells other contexts: nothing.

## Same word, different meaning

| Word | Rooms | Reservations | Payments | Notifications |
|---|---|---|---|---|
| Status | In service / out of service | Confirmed / Cancelled | Paid / Declined | Sent / Failed |
| Room | R8: Deluxe, nightly price, out of service | Just the number R8 on a reservation | Not used | Text in the message: "Room R8" |
| Cancel | Take a room off sale | Withdraw a reservation | Refund a deposit | Not used |

If Notifications could change a reservation's status to Failed, a failed SMS would overwrite Confirmed and Maya would lose R7 because of a message, which breaks rule R6 (a failed message must not lose a booking).

## Rule owners

| Card | Rule | Owner | Why this owner |
|---|---|---|---|
| 1 | A room never has two Confirmed reservations that share a night. | Reservations | It keeps every reservation of the room, so it can check them all together. |
| 2 | Check-out must be after check-in. | Reservations | It reads only the stay dates of the request, and Reservations owns stays and the booking rule. |
| 3 | Cancelling a reservation frees its nights for new bookings. | Reservations | Cancelling changes the reservation's own status, and Reservations keeps the reservations. |
| 4 | A room that is out of service is not offered to guests. | Rooms | Rooms keeps whether a room is in service, so only Rooms can decide it. |
| 5 | The nightly price depends on the room type (Deluxe costs more than Standard). | Rooms | Rooms keeps each room's type and nightly price. |
| 6 | The deposit is 30% of the stay's price. | Payments | Money rules belong to Payments; it asks Rooms for the nightly price. |
| 7 | Only a Paid deposit can be refunded. | Payments | Payments keeps every deposit and its status (Paid or Declined). |
| 8 | A failed message is recorded as Failed and never changes the reservation. | Notifications | Notifications keeps the messages and their status; it has no access to reservations. |

## Between contexts

### Reservations → Notifications: ReservationConfirmed (event)

Sent when: a reservation is saved as Confirmed.

Fields:
- reservationId: R101
- guestEmail: maya@example.com
- roomNumber: R7
- checkIn: 2026-11-12
- checkOut: 2026-11-14

Reservations does not wait for an answer. A failed message never changes R101 (rule R6).

Never crosses: Reservations' list of reservations. Notifications never reads it.

### Reservations → Rooms: which rooms exist? (question)

Sent when: a guest searches free rooms or books, and Reservations needs the list of rooms.

What Reservations sends: only the question, no data.

The answer and its fields: a list of rooms, each with roomNumber and roomType (R7 Standard, R8 Deluxe).

Never crosses: Reservations' own reservations. Rooms never reads them. Reservations never changes a room's price or in-service flag.

### Reservations → Payments: take a deposit (question, from P17)

Sent when: a guest books and a deposit must be taken (from P17).

What Reservations sends: reservationId (R101) and amount (30% of the stay's price).

The answer and its fields: status, Paid or Declined.

Never crosses: the reservation's status or guest details beyond the id. Payments never changes a reservation; it only answers Paid or Declined.
