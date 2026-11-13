# Quality scenarios

## Stakeholders

| Who | What they care about |
|---|---|
| Guest | Finds a free room quickly; never loses a booking |
| Receptionist | Never promises one room twice; can cancel at the desk |
| Hotel manager (domain expert) | Knows the rules; wants deposits and SMS soon |
| Development team | Can change one area without breaking the others |
| Email/SMS and payment providers | Outside systems that can be slow or down |

## S1 · Performance: many guests search at once

| Part | S1 |
|---|---|
| Source | 200 guests |
| Stimulus | They search free rooms for 12–14 Nov |
| Environment | Within one minute, during normal operation |
| Artifact | The free-room search in BookingService |
| Response | Every guest gets the correct free rooms |
| Measure | p95 response ≤ 1 second |
| Checked in | P08 (cache), P23 (load test) |

## S2 · Data integrity: no double booking

| Part | S2 |
|---|---|
| Source | 100 pairs of guests |
| Stimulus | Each pair books the same room and nights at the same moment |
| Environment | Normal operation, many requests at once |
| Artifact | BookingService.Reserve and the reservation store |
| Response | Exactly one Confirmed reservation per pair; the other guest is refused |
| Measure | 0 double bookings out of 100 pairs |
| Checked in | P09 |

## S3 · Availability: a failed message keeps the booking

| Part | S3 |
|---|---|
| Source | The email provider |
| Stimulus | It stops answering |
| Environment | For 10 minutes, during normal bookings |
| Artifact | BookingService.Reserve and the notification sender |
| Response | Every valid booking is still Accepted and saved; the failed message is reported |
| Measure | 0 lost bookings; every failed message is counted |
| Checked in | P04 (R6 test, done) and P10 |

## S4 · Security: a guest cannot read another guest's reservation

| Part | S4 |
|---|---|
| Source | A guest |
| Stimulus | They try to read another guest's reservation by its id |
| Environment | 100 attempts, while logged in as themselves |
| Artifact | Viewing a reservation by id |
| Response | The app refuses every attempt |
| Measure | 0 reservations shown to the wrong guest in 100 attempts |
| Checked in | P21 |

## S5 · Modifiability: adding SMS confirmations

| Part | S5 |
|---|---|
| Source | The hotel manager |
| Stimulus | Asks for SMS confirmations as well as email |
| Environment | During development, with the 8 tests passing |
| Artifact | BookingService and the notification senders |
| Response | A new sender class is added; BookingService is not edited |
| Measure | 0 edits to BookingService; at most 2 files change |
| Checked in | P04 (SMS sender, done) |

## S6 · Sustainability: less energy per search

| Part | S6 |
|---|---|
| Source | The development team |
| Stimulus | Runs the same 1,000 searches before and after a change |
| Environment | Same machine, same data |
| Artifact | The free-room search |
| Response | The changed version uses less energy for the same results |
| Measure | At least 20% fewer watt-hours per 1,000 completed searches |
| Checked in | P08, P24 |
