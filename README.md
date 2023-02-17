const express = require('express');
const mongoose = require('mongoose');

const app = express();
const port = 3000;

mongoose.connect('mongodb://localhost:27017/geolocation', { useNewUrlParser: true, useUnifiedTopology: true });

const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  phone_number: String,
  location: {
    type: { type: String },
    coordinates: [Number]
  }
});

const User = mongoose.model('User', userSchema);

app.use(express.json());

// Add User API
app.post('/users', (req, res) => {
  const user = new User(req.body);
  user.save((err, user) => {
    if (err) {
      res.status(500).send(err);
    } else {
      res.status(201).send(user);
    }
  });
});

// Get User API based on nearby geolocation coordinates
app.get('/users/nearby', (req, res) => {
  const { lng, lat, maxDistance } = req.query;
  User.find({
    location: {
      $near: {
        $geometry: {
          type: 'Point',
          coordinates: [lng, lat]
        },
        $maxDistance: maxDistance
      }
    }
  }, (err, users) => {
    if (err) {
      res.status(500).send(err);
    } else {
      res.send(users);
    }
  });
});

// Get User API based on partial matching name
app.get('/users', (req, res) => {
  const { name } = req.query;
  User.find({ name: { $regex: new RegExp(name, 'i') } }, (err, users) => {
    if (err) {
      res.status(500).send(err);
    } else {
      res.send(users);
    }
  });
});

// Update User API
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  User.findByIdAndUpdate(id, req.body, { new: true }, (err, user) => {
    if (err) {
      res.status(500).send(err);
    } else if (!user) {
      res.status(404).send('User not found');
    } else {
      res.send(user);
    }
  });
});

// Delete User API
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  User.findByIdAndDelete(id, (err, user) => {
    if (err) {
      res.status(500).send(err);
    } else if (!user) {
      res.status(404).send('User not found');
    } else {
      res.send('User deleted successfully');
    }
  });
});

app.listen(port, () => {
  console.log(`Server started at http://localhost:${port}`);
});
