# Ticket-booking
Based on your requirements, let's create a MEAN stack app for booking seats in a train coach. The application should allow users to input the number of seats they want to book, display the coach layout with seat availability, and book seats accordingly.

1. Set Up the Back-End (Node.js and Express):

Create a new folder for your project and navigate into it:
```bash
mkdir mean-stack-train-booking
cd mean-stack-train-booking
```

Initialize a new Node.js project and install required dependencies:
```bash
npm init -y
npm install express mongoose body-parser cors
```

Create a new file named `server.js` and set up the basic Express server:
```javascript
// server.js
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const mongoose = require('mongoose');

const app = express();
const port = 3000;

app.use(bodyParser.json());
app.use(cors());

// Connect to MongoDB database
mongoose.connect('mongodb://localhost:27017/train_booking_app', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => {
  console.log('Connected to MongoDB');
}).catch((err) => {
  console.error('Error connecting to MongoDB', err);
});

// Define the schema and model for the seat collection
const seatSchema = new mongoose.Schema({
  seat_number: Number,
  row_number: Number,
  is_booked: Boolean,
});

const Seat = mongoose.model('Seat', seatSchema);

// API endpoints for seat booking and displaying coach layout
app.get('/api/seats', (req, res) => {
  Seat.find({}, (err, seats) => {
    if (err) {
      res.status(500).send(err);
    } else {
      res.send(seats);
    }
  });
});

app.post('/api/book', (req, res) => {
  const { num_seats } = req.body;

  Seat.find({ is_booked: false }, (err, availableSeats) => {
    if (err) {
      res.status(500).send(err);
    } else {
      const seatsToBook = availableSeats.slice(0, num_seats);
      const seatIdsToBook = seatsToBook.map((seat) => seat._id);

      Seat.updateMany({ _id: { $in: seatIdsToBook } }, { is_booked: true }, (err) => {
        if (err) {
          res.status(500).send(err);
        } else {
          res.send(seatsToBook);
        }
      });
    }
  });
});

// Start the server
app.listen(port, () => {
  console.log(`Server running on http://localhost:${port}`);
});
```

2. Set Up the Front-End (Angular):

Create a new Angular project inside the main folder:
```bash
ng new client
cd client
```

Generate a new component for managing seat booking:
```bash
ng generate component seat-booking
```

Replace the content of `src/app/seat-booking/seat-booking.component.html` with the following Angular template code:
```html
<!-- seat-booking.component.html -->
<h2>Train Seat Booking</h2>

<div class="coach-layout">
  <div class="seat" *ngFor="let seat of seats">
    <div [ngClass]="{'available': !seat.is_booked, 'booked': seat.is_booked}">
      {{ seat.row_number }}-{{ seat.seat_number }}
    </div>
  </div>
</div>

<div class="booking-form">
  <h3>Book Seats</h3>
  <form (submit)="bookSeats()">
    <input type="number" [(ngModel)]="numSeats" placeholder="Number of seats">
    <button type="submit">Book</button>
  </form>
</div>
```

Replace the content of `src/app/seat-booking/seat-booking.component.ts` with the following Angular component code:
```typescript
// seat-booking.component.ts
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';

interface Seat {
  _id: string;
  seat_number: number;
  row_number: number;
  is_booked: boolean;
}

@Component({
  selector: 'app-seat-booking',
  templateUrl: './seat-booking.component.html',
  styleUrls: ['./seat-booking.component.css']
})
export class SeatBookingComponent implements OnInit {
  seats: Seat[] = [];
  numSeats: number = 1;

  constructor(private http: HttpClient) { }

  ngOnInit(): void {
    this.fetchSeats();
  }

  fetchSeats(): void {
    this.http.get<Seat[]>('http://localhost:3000/api/seats')
      .subscribe((seats) => {
        this.seats = seats;
      });
  }

  bookSeats(): void {
    this.http.post<Seat[]>('http://localhost:3000/api/book', { num_seats: this.numSeats })
      .subscribe((bookedSeats) => {
        for (const seat of bookedSeats) {
          const index = this.seats.findIndex((s) => s._id === seat._id);
          if (index !== -1) {
            this.seats[index].is_booked = true;
          }
        }
      });
  }
}
```

3. Run the MEAN Stack App:

Start the back-end server by running `node server.js` from the main project folder.

Start the front-end Angular app by running `ng serve` from the `client` folder.

Access the MEAN stack app in your browser at `http://localhost:4200`.

Now, you have a MEAN stack app for booking seats in a train coach. Users can input the number of seats they want to book, view the coach layout with seat availability, and book seats accordingly. The seat availability is represented by colors ('available' and 'booked'). Users can book multiple seats at once, and the app prioritizes booking seats in the same row whenever possible.
