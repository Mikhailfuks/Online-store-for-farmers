const mongoose = require('mongoose');

const productSchema = new mongoose.Schema({
    name: { type: String, required: true },
    price: { type: Number, required: true },
    description: { type: String, required: true },
    stock: { type: Number, required: true },
}, { timestamps: true });

module.exports = mongoose.model('Product', productSchema);

const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    password: { type: String, required: true },
}, { timestamps: true });

module.exports = mongoose.model('User', userSchema);

// index.js
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const cors = require('cors');
require('dotenv').config();

const Product = require('./models/Product');
const User = require('./models/User');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(cors());
app.use(bodyParser.json());

// Connect to MongoDB
mongoose.connect(process.env.MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected successfully!'))
.catch(err => console.error('MongoDB connection error:', err));

// Route to add a new product
app.post('/products', async (req, res) => {
    const { name, price, description, stock } = req.body;
    try {
        const product = new Product({ name, price, description, stock });
        await product.save();
        res.status(201).json(product);
    } catch (error) {
        res.status(400).json({ message: 'Error adding product', error });
    }
});

// Route to list all products
app.get('/products', async (req, res) => {
    try {
        const products = await Product.find();
        res.status(200).json(products);
    } catch (error) {
        res.status(500).json({ message: 'Error fetching products', error });
    }
});

// Route to register a new user
app.post('/register', async (req, res) => {
    const { username, password } = req.body;
    try {
        const user = new User({ username, password });
        await user.save();
        res.status(201).json(user);
    } catch (error) {


        res.status(400).json({ message: 'Error registering user', error });
    }
});

// Start the server
app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Online Store for Farmers</title>
    <style>
        body { font-family: Arial, sans-serif; }
        .product { margin: 15px 0; }
    </style>
</head>
<body>
    <h1>Online Store for Farmers</h1>

    <h2>Products</h2>
    <div id="products"></div>

    <h2>Add Product</h2>
    <input id="name" placeholder="Name" /><br />
    <input id="price" placeholder="Price" /><br />
    <input id="description" placeholder="Description" /><br />
    <input id="stock" placeholder="Stock" /><br />
    <button onclick="addProduct()">Add Product</button>

    <script>
        async function fetchProducts() {
            const response = await fetch('http://localhost:3000/products');
            const products = await response.json();
            const productsDiv = document.getElementById('products');
            productsDiv.innerHTML = '';
            products.forEach(product => {
                const productDiv = document.createElement('div');
                productDiv.className = 'product';
                productDiv.innerHTML = `<strong>${product.name}</strong> - $${product.price} <br />${product.description} <br />Stock: ${product.stock}<br />`;
                productsDiv.appendChild(productDiv);
            });
        }

        async function addProduct() {
            const name = document.getElementById('name').value;
            const price = document.getElementById('price').value;
            const description = document.getElementById('description').value;
            const stock = document.getElementById('stock').value;

            const response = await fetch('http://localhost:3000/products', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({name, price, description, stock})
            });

            if (response.ok) {
                fetchProducts(); // Refresh product list
                document.getElementById('name').value = '';
                document.getElementById('price').value = '';
                document.getElementById('description').value = '';
                document.getElementById('stock').value = '';
            }
        }

        // Initial fetch
        fetchProducts();
    </script>
</body>
</html>
