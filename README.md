# Restaurant Reservation

A full-stack table-booking app for restaurants. Guests pick a time and party size on an interactive floor plan, and the app recommends the table that fits them best.

**Stack:** Java 25 · Spring Boot 3.5 · H2 · React 19 · Vite · REST

<!-- Add a screenshot or GIF of the floor plan here, e.g. docs/screenshot.png -->

## How it works

1. **Search.** Choose a date, time, party size and zone (indoor, terrace or private room). Optionally ask for a quiet corner, a window seat or a spot near the kids' play area.
2. **Recommendation.** The backend scores every free table by how closely its capacity matches the party and how many of the requested features it has. The best match pulses green on the floor plan.
3. **Booking.** Accept the recommendation or click any other free table, enter a name and email, and confirm.
4. **Live floor plan.** The booked table turns red straight away. Bookings last two hours and expire automatically after that.

On startup the app generates random existing bookings, so the floor plan starts out partly occupied.

## Features

- SVG floor plan with 19 tables across 3 zones. Tables outside the selected zone are dimmed, not hidden, so the whole room stays visible.
- Score-based recommendation engine with unit tests
- Double-booking prevention: overlapping times are checked on the server, and party size is validated against table capacity on both client and server
- A background job marks finished bookings as expired every 60 seconds
- The booking confirmation suggests a meal from the public TheMealDB API

## Architecture

```
frontend/   React + Vite single-page app (floor plan, filters, booking flow)
backend/    Spring Boot REST API
  controller/   /api/tables, /api/recommendations, /api/reservations, /api/meals
  service/      recommendation scoring, availability, expiry scheduler
  domain/       RestaurantTable, Reservation, Zone, TableFeature (JPA entities)
  seeder/       random demo bookings on startup
```

The data lives in an in-memory H2 database, so it needs no setup and resets on every restart.

## Running it

**Requirements:** Java 25 JDK and Node.js 22 or newer.

### One server (production-style build)

```bash
cd frontend && npm install && npm run build
cp -r dist ../backend/src/main/resources/static/
cd ../backend && ./mvnw spring-boot:run
```

Open <http://localhost:8080>.

### Development, with hot reload

```bash
# Terminal 1
cd backend && ./mvnw spring-boot:run

# Terminal 2 (sends /api requests on to port 8080)
cd frontend && npm install && npm run dev
```

Open <http://localhost:5173>. The database console is at <http://localhost:8080/h2-console>, with the JDBC URL `jdbc:h2:mem:restaurant`.

### Tests

```bash
cd backend && ./mvnw test
```

## Design notes

[`docs/DEVLOG.md`](docs/DEVLOG.md) records the assumptions, the bugs that were hard to fix (such as wrap-around past midnight in time-overlap checks) and how they were solved. [`docs/PLAN.md`](docs/PLAN.md) has the original implementation plan.

## Roadmap

- [ ] Persistent database (PostgreSQL) in place of in-memory H2
- [ ] Staff view for managing bookings and editing the floor plan
- [ ] Email confirmation and cancellation links
- [ ] Bookings that run past midnight
- [ ] Deployment with a public demo link
