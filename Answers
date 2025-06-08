Task 1: Express.js Setup
  mkdir express-api && cd express-api
  npm init -y
  npm install express body-parser uuid dotenv

Task 2: RESTful API Routes
// productModel.js

const { v4: uuidv4 } = require('uuid');

let products = [];

module.exports = {
  getAll: () => products,
  getById: (id) => products.find(p => p.id === id),
  create: (data) => {
    const newProduct = { id: uuidv4(), ...data };
    products.push(newProduct);
    return newProduct;
  },
  update: (id, data) => {
    const index = products.findIndex(p => p.id === id);
    if (index !== -1) {
      products[index] = { ...products[index], ...data };
      return products[index];
    }
    return null;
  },
  remove: (id) => {
    const index = products.findIndex(p => p.id === id);
    if (index !== -1) {
      return products.splice(index, 1)[0];
    }
    return null;
  }
};

// productController
const Product = require('../models/productModel');
const { NotFoundError } = require('../utils/errors');

exports.getAllProducts = (req, res) => {
  let result = Product.getAll();

  // Filtering
  if (req.query.category) {
    result = result.filter(p => p.category === req.query.category);
  }

  // Pagination
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || result.length;
  const start = (page - 1) * limit;
  const end = page * limit;
  const paginated = result.slice(start, end);

  res.json({ total: result.length, data: paginated });
};

exports.getProductById = (req, res, next) => {
  const product = Product.getById(req.params.id);
  if (!product) return next(new NotFoundError('Product not found'));
  res.json(product);
};

exports.createProduct = (req, res) => {
  const product = Product.create(req.body);
  res.status(201).json(product);
};

exports.updateProduct = (req, res, next) => {
  const updated = Product.update(req.params.id, req.body);
  if (!updated) return next(new NotFoundError('Product not found'));
  res.json(updated);
};

exports.deleteProduct = (req, res, next) => {
  const deleted = Product.remove(req.params.id);
  if (!deleted) return next(new NotFoundError('Product not found'));
  res.json(deleted);
};

exports.searchProducts = (req, res) => {
  const { name } = req.query;
  const result = Product.getAll().filter(p => p.name.toLowerCase().includes(name.toLowerCase()));
  res.json(result);
};

exports.getStats = (req, res) => {
  const products = Product.getAll();
  const stats = {};
  products.forEach(p => {
    stats[p.category] = (stats[p.category] || 0) + 1;
  });
  res.json(stats);
};


// productRoutes
const express = require('express');
const router = express.Router();
const controller = require('../controllers/productController');
const validateProduct = require('../middleware/validateProduct');
const auth = require('../middleware/auth');

router.use(auth); // All routes require API key

router.get('/', controller.getAllProducts);
router.get('/search', controller.searchProducts);
router.get('/stats', controller.getStats);
router.get('/:id', controller.getProductById);
router.post('/', validateProduct, controller.createProduct);
router.put('/:id', validateProduct, controller.updateProduct);
router.delete('/:id', controller.deleteProduct);

module.exports = router;



Task 3: Middleware Implementation
//logger
module.exports = (req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next();
};

//auth
module.exports = (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  if (!apiKey || apiKey !== process.env.API_KEY) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  next();
};

//validateProduct
const { ValidationError } = require('../utils/errors');

module.exports = (req, res, next) => {
  const { name, description, price, category, inStock } = req.body;
  if (!name || !description || typeof price !== 'number' || !category || typeof inStock !== 'boolean') {
    return next(new ValidationError('Invalid product fields'));
  }
  next();
};



Task 4: Error Handling
class NotFoundError extends Error {
  constructor(message) {
    super(message);
    this.statusCode = 404;
  }
}

class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.statusCode = 400;
  }
}

module.exports = { NotFoundError, ValidationError };

module.exports = (err, req, res, next) => {
  const status = err.statusCode || 500;
  res.status(status).json({ error: err.message || 'Server Error' });
};


