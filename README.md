# Smart Barbershop

A full-stack web app for running a barbershop: customers book appointments, get hairstyle recommendations from an in-browser face-shape analyzer, and buy products online, while the barber manages everything from a dashboard.

Built as a team project for COE 416 (Software Engineering) at the Lebanese American University, Spring 2026.

<!-- Add a screenshot or demo GIF here, e.g. ![Smart Barbershop demo](docs/demo.gif) -->

## Features

**Booking**
- Customers pick a service and a time slot within the barber's working hours
- Overlapping bookings are rejected
- Customers can cancel up to a configurable number of hours before the appointment (`CANCEL_CUTOFF_HOURS`)
- Pending appointments whose time has passed are cancelled automatically
- Printable booking receipts

**AI hairstyle recommendations**
- Uses MediaPipe Face Landmarker (478 facial landmarks), running entirely in the browser via WebAssembly, so the photo never leaves the user's device
- Classifies the face into one of 5 shapes (oval, round, square, oblong, diamond) from facial proportions and recommends matching hairstyles

**Online store**
- Product catalog, cart, and checkout
- Stock is reserved atomically, so two customers checking out at the same moment can't oversell the last item (see below)
- Short cancellation window after ordering, plus order history

**Barber dashboard**
- Appointments with filters and a calendar view
- Working-hours editor, inventory management, and order management
- Mark appointments and orders as paid

**Authentication**
- Passwords hashed with bcrypt
- Sessions stored as signed JWTs in httpOnly cookies
- Role-based access: only barbers can reach the dashboard

## Tech Stack

| Layer | Technology |
| --- | --- |
| Framework | Next.js 16 (App Router), React 19 |
| Styling | Tailwind CSS 4 |
| Database | SQLite with Prisma 6 |
| Auth | jose (JWT), bcryptjs |
| Validation | zod |
| Face analysis | MediaPipe Tasks Vision (WebAssembly) |
| Testing | Vitest (including concurrent-checkout tests) |

## How checkout prevents overselling

A naive checkout reads the stock, checks it, and then decrements it. If two requests arrive at the same moment, both can pass the check before either writes, and the stock goes negative.

Here, the check and the decrement are a single conditional update inside a transaction (`lib/orders.js`):

```js
const { count } = await tx.product.updateMany({
  where: { id: item.productId, stock: { gte: item.quantity } },
  data: { stock: { decrement: item.quantity } },
})
if (count === 0) throw new OrderError(`Not enough stock for ${product.name}.`, 409)
```

If any item in the cart can't be reserved, the whole transaction rolls back. Quantities are validated with zod (whole numbers from 1 to 100), and prices always come from the database, never from the client.

## Limitations and Next Steps

- Payments are recorded manually by the barber; no payment gateway is integrated yet
- Hairstyle previews use curated reference images rather than generated images
- Heart-shaped faces are currently classified as oval
- Booking uses a check-then-insert for overlapping slots; it should use the same atomic approach as checkout

## Team

Ronald Baaklini, Nicolas Zakkak, Karl Al Chikhani, Hussein Hojeij

## Source Code

The source code is private because this is a university course project. It is available on request: ronald.baaklini@gmail.com
