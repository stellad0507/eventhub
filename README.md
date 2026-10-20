# EventHub

A backend API for a simplified event ticketing platform, built with Django, Django REST Framework, ViewSets and a DefaultRouter. Users can browse events, reserve seats and cancel reservations.

## How to run

```bash
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

The API is then available at `http://127.0.0.1:8000/api/`. Every request is logged to the terminal by `RequestLoggingMiddleware`, for example `POST /api/reservations/ - 201 - 0.01s`.

## Endpoints

### Events

| Method | URL | What it does |
|---|---|---|
| GET | `/api/events/` | List events. Optional filters: `?status=upcoming`, `?venue=bangalore` (case-insensitive contains) |
| POST | `/api/events/` | Create an event |
| GET | `/api/events/{id}/` | Retrieve one event (includes `reservations_count`, the confirmed reservations) |
| PUT / PATCH | `/api/events/{id}/` | Update an event |
| DELETE | `/api/events/{id}/` | Delete an event |

### Reservations

| Method | URL | What it does |
|---|---|---|
| GET | `/api/reservations/` | List reservations. Optional filter: `?event_id=1` |
| POST | `/api/reservations/` | Reserve seats. Deducts them from the event's `available_seats`. Returns 400 for overbooking, or if the event is completed or cancelled |
| GET | `/api/reservations/{id}/` | Retrieve one reservation |
| PUT / PATCH / DELETE | `/api/reservations/{id}/` | Standard update / delete |
| POST | `/api/reservations/{id}/cancel/` | Cancel a reservation and restore its seats to the event. Returns 400 if it is already cancelled |

## Design decision

Seat deduction happens inside `ReservationSerializer.create()`, next to the creation of the `Reservation`. Validation (`validate()`) checks event status and seat availability first. This keeps the rule "a reservation exists if and only if its seats were deducted" in a single place, instead of splitting it between the view and the serializer.

The trade-off is concurrency: two simultaneous requests for the last seat could both pass validation. A production system would wrap the check and write in `transaction.atomic()` with `select_for_update()` on the event row. I kept it out here as the brief scopes it out.
