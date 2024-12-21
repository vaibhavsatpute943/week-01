const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const bodyParser = require('body-parser');
const crypto = require('crypto');

const app = express();
app.use(cors());
app.use(express.json());
app.use(bodyParser.urlencoded({ extended: true }));

mongoose.connect(
  'mongodb://localhost:27017/up_clone',
  {
    useNewUrlParser: true,
    useUnifiedTopology: true
  }
)
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error(err));

// Define User Schema
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
  upi_id: { type: String, unique: true },
  balance: { type: Number, default: 1000 }
});

// Create User Model
const User = mongoose.model('User', userSchema);

// Define Transaction Schema
const transactionSchema = new mongoose.Schema({
  sender_upi_id: { type: String, required: true },
  receiver_upi_id: { type: String, required: true },
  amount: { type: Number, required: true },
  timestamp: { type: Date, default: Date.now }
});

// Create Transaction Model
const Transaction = mongoose.model('Transaction', transactionSchema);

// Function to generate a unique UPI ID
const generateUPI = () => {
  const randomId = crypto.randomBytes(4).toString('hex'); // Generates a random 8-character ID
  return `${randomId}@fastpay`;
};

// Signup Route
app.post('/api/signup', async (req, res) => {
  try {
    const { name, email, password } = req.body;

    // Check if user already exists
    let user = await User.findOne({ email });
    if (user) {
      return res.status(400).send({ message: 'User already exists' });
    }

    // Generate UPI ID
    const upi_id = generateUPI();
    const balance = 1000;

    // Create new user
    user = new User({ name, email, password, upi_id, balance });
    await user.save();

    res.status(201).send({ message: 'User registered successfully!', upi_id });
  } catch (error) {
    console.error(error);
    res.status(500).send({ message: 'Server error' });
  }
});

// Get User Details Route
app.get('/api/user/:upi_id', async (req, res) => {
  try {
    const { upi_id } = req.params;

    const user = await User.findOne({ upi_id });
    if (!user) {
      return res.status(404).send({ message: 'User not found' });
    }

    res.status(200).send(user);
  } catch (error) {
    console.error(error);
    res.status(500).send({ message: 'Server error' });
  }
});

// Transaction Route
app.post('/api/transaction', async (req, res) => {
  try {
    const { sender_upi_id, receiver_upi_id, amount } = req.body;

    // Validate users
    const sender = await User.findOne({ upi_id: sender_upi_id });
    const receiver = await User.findOne({ upi_id: receiver_upi_id });

    if (!sender || !receiver) {
      return res.status(404).send({ message: 'Sender or receiver not found' });
    }

    if (sender.balance < amount) {
      return res.status(400).send({ message: 'Insufficient balance' });
    }

    // Update balances
    sender.balance -= amount;
    receiver.balance += amount;

    await sender.save();
    await receiver.save();

    // Record transaction
    const transaction = new Transaction({ sender_upi_id, receiver_upi_id, amount });
    await transaction.save();

    res.status(201).send({ message: 'Transaction successful' });
  } catch (error) {
    console.error(error);
    res.status(500).send({ message: 'Server error' });
  }
});

// Get Transaction History Route
app.get('/api/transactions/:upi_id', async (req, res) => {
  try {
    const { upi_id } = req.params;

    const transactions = await Transaction.find({
      $or: [
        { sender_upi_id: upi_id },
        { receiver_upi_id: upi_id }
      ]
    }).sort({ timestamp: -1 });

    res.status(200).send(transactions);
  } catch (error) {
    console.error(error);
    res.status(500).send({ message: 'Server error' });
  }
});

// Start the server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
