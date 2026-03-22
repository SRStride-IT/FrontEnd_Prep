# Calendar App – Schedule Timing Slots

## Goal
Allow users to create and view available time slots in a calendar app.

## Recommended Slot Rules
- Slot duration: 15, 30, or 60 minutes.
- Working hours: 09:00 to 18:00.
- Buffer time between meetings: 5 or 10 minutes.
- Prevent overlapping bookings.
- Respect timezone for each user.

## Data Model (Example)
```ts
export interface TimeSlot {
  id: string;
  start: string; // ISO date-time
  end: string;   // ISO date-time
  status: 'available' | 'booked' | 'blocked';
  title?: string;
}
```

## Slot Generation Logic (TypeScript)
```ts
import { addMinutes, isBefore } from 'date-fns';

export function generateSlots(
  dayStart: Date,
  dayEnd: Date,
  slotDurationMinutes = 30,
  bufferMinutes = 0,
): Array<{ start: Date; end: Date }> {
  const slots: Array<{ start: Date; end: Date }> = [];
  let current = new Date(dayStart);

  while (isBefore(addMinutes(current, slotDurationMinutes), dayEnd) ||
         addMinutes(current, slotDurationMinutes).getTime() === dayEnd.getTime()) {
    const end = addMinutes(current, slotDurationMinutes);
    slots.push({ start: new Date(current), end });
    current = addMinutes(end, bufferMinutes);
  }

  return slots;
}
```

## Overlap Validation
```ts
export function hasOverlap(
  candidateStart: Date,
  candidateEnd: Date,
  existing: Array<{ start: Date; end: Date }>,
): boolean {
  return existing.some(
    slot => candidateStart < slot.end && candidateEnd > slot.start,
  );
}
```

## API Suggestions
- `GET /api/slots?date=YYYY-MM-DD&timezone=Asia/Kolkata`
- `POST /api/slots` to create availability windows.
- `POST /api/bookings` to reserve a slot.
- `PATCH /api/slots/:id` to block/unblock a slot.

## UI Flow
1. User picks date in calendar.
2. App fetches slots for that date.
3. Available slots are shown as buttons/chips.
4. On click, booking confirmation modal opens.
5. After booking, slot status changes to `booked`.

## Edge Cases
- Daylight Saving Time transitions.
- Same user opening app in multiple tabs (avoid double booking).
- Past time slots should be disabled.
- Holidays/non-working days should be blocked.

## Interview-Friendly Summary
"In a calendar app, I design schedule timing slots by generating fixed-duration intervals inside working hours, applying buffer time, validating overlap before booking, and storing all timestamps in ISO + timezone-aware format. On the UI, slots are fetched per selected date and instantly updated after booking to avoid conflicts."
