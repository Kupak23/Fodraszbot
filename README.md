# Fodraszbot
require('dotenv').config();
const express = require('express');
const bodyParser = require('body-parser');
const { initDB, saveBooking } = require('./db');
const products = require('./products.json');

const app = express();
app.use(bodyParser.json());

const VERIFY_TOKEN = process.env.VERIFY_TOKEN;

// Facebook webhook verify
app.get('/webhook', (req, res) => {
  const mode = req.query['hub.mode'];
  const token = req.query['hub.verify_token'];
  const challenge = req.query['hub.challenge'];
  if (mode && token && token === VERIFY_TOKEN) {
    res.status(200).send(challenge);
  } else {
    res.sendStatus(403);
  }
});

// Facebook webhook üzenet fogadás
app.post('/webhook', (req, res) => {
  const body = req.body;
  if (body.object === 'page') {
    body.entry.forEach(entry => {
      const webhookEvent = entry.messaging[0];
      const senderId = webhookEvent.sender.id;
      const message = webhookEvent.message?.text;
      if (message) handleMessage(senderId, message);
    });
    res.status(200).send('EVENT_RECEIVED');
  } else {
    res.sendStatus(404);
  }
});

// Egyszerű üzenetfeldolgozó
function handleMessage(senderId, message) {
  message = message.toLowerCase();
  if (message.includes('időpont')) {
    console.log(`${senderId} szeretne időpontot`);
  } else if (message.includes('ár')) {
    console.log(`${senderId} árlistát kér`);
  } else if (message.includes('fordítás')) {
    console.log(`${senderId} fordítást kér`);
  } else {
    console.log(`${senderId} egyéb üzenet: ${message}`);
  }
}

// DB inicializálása
initDB();

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server is running on port ${PORT}`));
