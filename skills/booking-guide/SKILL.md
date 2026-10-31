---
name: booking-guide
description: Use when the user wants to book, cancel, or check UCSD study room reservations. Trigger on "study room", "방 예약", "예약 취소", "스터디룸", "예약 조" "Price Center", or any room booking intent.
---

# UCSD Study Room Booking Guide

You have access to MCP tools for managing UCSD Price Center Study Room reservations.

## Available Tools

- `search_rooms(date, start_time, end_time)` — Search available rooms
- `book_room(date, start_time, end_time, room_name)` — Book a specific room
- `my_events()` — List current reservations
- `cancel_reservation(reason, reservation_id?, date?, room_name?)` — Cancel a reservation (pass `reservation_id`, or a `YYYY-MM-DD` date)
- `login(username, password)` — Authenticate (UCSD SSO + Duo Push, runs headlessly — no browser window)

## Booking Flow

1. Confirm date and time with the user (date: `YYYY-MM-DD`, time: `HH:MM` 24h format)
2. Call `search_rooms` to find available rooms
3. Present the available rooms and ask the user which one to book
4. Call `book_room` with the selected room name (exact match required, e.g., "Price Center Study Room 3")

## Cancellation Flow

1. Call `my_events` to show current reservations
2. Ask the user which reservation to cancel and why
3. Reason MUST be one of: Bad Weather, Changed Date, Changed Location, Lack of Funding, Lack of Interest, Lack of Resources, Lack of Time to Plan, Other
4. Call `cancel_reservation` with the selected reservation's `reservation_id` (from `my_events`) and reason
5. If it returns multiple matches (`needs_selection`), ask the user which one and call again with that `reservation_id`. Never guess between them

## Important Rules

- **Search has no time limit** — `search_rooms` can query any time range. Do NOT split searches into 4-hour chunks.
- **Booking has a 4-hour max** — `book_room` is limited to 4 hours per reservation per day. If the user wants more than 4 hours, split into multiple bookings.

## Error Handling

- If a tool returns "Session expired", call `login` first, then retry
- **Before calling `login`**, tell the user that login is starting: it runs in the background with no window and can take up to 90 seconds, and they need to approve the Duo Push on their phone. Without this message the user sees nothing while waiting.
- If `login` fails with "SSO login failed: ...", show the user the reason as-is; do not retry with the same credentials
- Rooms are Price Center Study Room 1 through 8
