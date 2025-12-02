# Coffee-Bakery-store-full.
Full stack coffee shop with backend API
coffee-shop-full/
├── frontend/           (مشروعي)
│   ├── index.html
│   ├── css/
│   ├── js/            ( ملفات جديدة)
│   └── images/
├── backend/           (المشروع الجديد)
└── README.md
{
  "name": "coffee-bakery-backend",
  "version": "1.0.0",
  "description": "Backend API for Coffee & Bakery Store",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "seed": "node seeds/productSeeder.js"
  },
  "keywords": ["coffee", "bakery", "store", "ecommerce", "api"],
  "author": "Coffee Shop",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0",
    "dotenv": "^16.0.3",
    "cors": "^2.8.5",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.22"
  }
}










const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const dotenv = require('dotenv');
const path = require('path');

// Load environment variables
dotenv.config();

// Initialize Express app
const app = express();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Database connection
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/coffee-bakery', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log(' Connected to MongoDB'))
.catch(err => console.error('
MongoDB connection error:', err));

// Import models
const Product = require('./models/Product');
const Order = require('./models/Order');

// Basic route
app.get('/api', (req, res) => {
  res.json({
    message: ' Coffee & Bakery Store API',
    version: '1.0.0',
    endpoints: {
      products: '/api/products',
      orders: '/api/orders',
      categories: '/api/products/categories'
    }
  });
});

// Get all products
app.get('/api/products', async (req, res) => {
  try {
    const { category } = req.query;
    let filter = {};
    
    if (category) filter.category = category;
    
    const products = await Product.find(filter);
    res.json({ success: true, data: products });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// Get single product
app.get('/api/products/:id', async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (!product) return res.status(404).json({ success: false, error: 'Product not found' });
    res.json({ success: true, data: product });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// Get categories
app.get('/api/products/categories', async (req, res) => {
  try {
    const categories = await Product.distinct('category');
    res.json({ success: true, data: categories });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// Create order
app.post('/api/orders', async (req, res) => {
  try {
    const order = await Order.create(req.body);
    res.status(201).json({ 
      success: true, 
      message: 'Order placed successfully!',
      data: order 
    });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// Get order by ID
app.get('/api/orders/:id', async (req, res) => {
  try {
    const order = await Order.findById(req.params.id);
    if (!order) return res.status(404).json({ success: false, error: 'Order not found' });
    res.json({ success: true, data: order });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
});

// Start server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`
  Server running on http://localhost:${PORT}`);
});









PORT=5000
MONGODB_URI=mongodb://localhost:27017/coffee-bakery








const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  nameAr: {
    type: String,
    required: true,
    trim: true
  },
  description: {
    type: String,
    required: true
  },
  price: {
    type: Number,
    required: true,
    min: 0
  },
  category: {
    type: String,
    required: true,
    enum: ['coffee', 'bakery', 'dessert', 'drink']
  },
  type: {
    type: String,
    required: true
  },
  image: {
    type: String,
    default: 'default.jpg'
  },
  available: {
    type: Boolean,
    default: true
  },
  popular: {
    type: Boolean,
    default: false
  }
}, {
  timestamps: true
});

module.exports = mongoose.model('Product', productSchema);












const mongoose = require('mongoose');

const orderSchema = new mongoose.Schema({
  customer: {
    name: {
      type: String,
      required: true,
      trim: true
    },
    phone: {
      type: String,
      required: true,
      trim: true
    },
    email: {
      type: String,
      trim: true
    },
    address: {
      type: String,
      required: true
    }
  },
  items: [{
    product: {
      type: String,
      required: true
    },
    name: {
      type: String,
      required: true
    },
    quantity: {
      type: Number,
      required: true,
      min: 1
    },
    price: {
      type: Number,
      required: true
    }
  }],
  subtotal: {
    type: Number,
    required: true
  },
  deliveryFee: {
    type: Number,
    default: 0
  },
  total: {
    type: Number,
    required: true
  },
  status: {
    type: String,
    enum: ['pending', 'confirmed', 'delivered', 'cancelled'],
    default: 'pending'
  },
  paymentMethod: {
    type: String,
    enum: ['cash', 'card'],
    default: 'cash'
  },
  notes: {
    type: String
  }
}, {
  timestamps: true
});

module.exports = mongoose.model('Order', orderSchema);


















const mongoose = require('mongoose');
const dotenv = require('dotenv');
const Product = require('../models/Product');

dotenv.config();

const products = [
  {
    name: "Espresso",
    nameAr: "إسبريسو",
    description: "Strong and rich coffee shot",
    descriptionAr: "شوت قهوة قوي وغني",
    price: 25,
    category: "coffee",
    type: "hot",
    image: "espresso.jpg",
    popular: true,
    available: true
  },
  {
    name: "Cappuccino",
    nameAr: "كابتشينو",
    description: "Espresso with steamed milk and foam",
    descriptionAr: "إسبريسو مع حليب مبخور ورغوة",
    price: 30,
    category: "coffee",
    type: "hot",
    image: "cappuccino.jpg",
    popular: true,
    available: true
  },
  {
    name: "Latte",
    nameAr: "لاتيه",
    description: "Espresso with lots of steamed milk",
    descriptionAr: "إسبريسو مع كمية كبيرة من الحليب المبخور",
    price: 32,
    category: "coffee",
    type: "hot",
    image: "latte.jpg",
    popular: false,
    available: true
  },
  {
    name: "Iced Coffee",
    nameAr: "قهوة مثلجة",
    description: "Cold brew coffee with ice",
    descriptionAr: "قهوة مخمرة باردة مع ثلج",
    price: 28,
    category: "drink",
    type: "cold",
    image: "iced-coffee.jpg",
    popular: true,
    available: true
  },
  {
    name: "Croissant",
    nameAr: "كرواسون",
    description: "Freshly baked French pastry",
    descriptionAr: "معجنات فرنسية طازجة",
    price: 20,
    category: "bakery",
    type: "pastry",
    image: "croissant.jpg",
    popular: true,
    available: true
  },
  {
    name: "Bagel",
    nameAr: "بيغل",
    description: "Fresh bagel with sesame seeds",
    descriptionAr: "بيغل طازج مع حبوب السمسم",
    price: 18,
    category: "bakery",
    type: "bread",
    image: "bagel.jpg",
    popular: false,
    available: true
  },
  {
    name: "Chocolate Cake",
    nameAr: "كيكة الشوكولاتة",
    description: "Rich chocolate cake with cream",
    descriptionAr: "كيكة شوكولاتة غنية مع كريمة",
    price: 45,
    category: "dessert",
    type: "cake",
    image: "chocolate-cake.jpg",
    popular: true,
    available: true
  },
  {
    name: "Chocolate Chip Cookie",
    nameAr: "كوكيز رقائق الشوكولاتة",
    description: "Freshly baked cookies with chocolate chips",
    descriptionAr: "كوكيز طازجة مع رقائق الشوكولاتة",
    price: 15,
    category: "dessert",
    type: "cookie",
    image: "cookie.jpg",
    popular: false,
    available: true
  },
  {
    name: "Muffin",
    nameAr: "مافن",
    description: "Blueberry muffin",
    descriptionAr: "مافن التوت البري",
    price: 22,
    category: "bakery",
    type: "pastry",
    image: "muffin.jpg",
    popular: true,
    available: true
  },
  {
    name: "Turkish Coffee",
    nameAr: "قهوة تركية",
    description: "Traditional Turkish coffee",
    descriptionAr: "قهوة تركية تقليدية",
    price: 35,
    category: "coffee",
    type: "hot",
    image: "turkish-coffee.jpg",
    popular: false,
    available: true
  }
];

async function seedDatabase() {
  try {
    await mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/coffee-bakery');
    console.log('Connected to MongoDB');
    
    // Clear existing products
    await Product.deleteMany({});
    console.log('Cleared existing products');
    
    // Insert new products
    await Product.insertMany(products);
    console.log(`Added ${products.length} products`);
    
    // Display added products
    const allProducts = await Product.find({});
    console.log('\n Products in database:');
    allProducts.forEach(p => {
      console.log(`- ${p.name} (${p.nameAr}): $${p.price}`);
    });
    
    mongoose.disconnect();
    console.log('\n Database seeded successfully!');
    
  } catch (error) {
    console.error('Error seeding database:', error);
    process.exit(1);
  }
}

seedDatabase();













