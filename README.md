# Documentacion_Express

A continuación te guío para crear el CRUD completo para la tabla de "productos" con los campos id, nombre, precio, y descripción. También incluiré la documentación para que puedas agregarla a tu archivo README.md.

# Pasos para la creacion del proyecto en express

## 1. **Instalar Node.js:** Asegúrate de tener Node.js instalado. Si no lo tienes, puedes descargarlo de nodejs.org.


## 2. **Inicializar el proyecto:** Abre una terminal y sigue estos pasos:
   
```bash
mkdir mi-proyecto-backend
cd mi-proyecto-backend
npm init -y
```

3. **Instalar Express:** Ahora, instala Express y algunas dependencias comunes:

```bash
npm install express morgan dotenv cors
```

- **Express:** el framework.
- **Morgan:** para el registro de solicitudes HTTP.
- **dotenv:** para manejar las variables de entorno.
- **cors:** para habilitar la comunicación con el frontend desde diferentes dominios.

## 4. **Estructura básica de carpetas:** A continuación te doy una estructura comúnmente utilizada para mantener un código limpio y organizado:

```bash
├── src
│   ├── config
│   │   └── database.js            # Configuración base de datos
│   ├── controllers
│   │   └── productController.js      # Controladores para las rutas
│   ├── models
│   │   └── productModel.js         # Modelos de base de datos
│   ├── routes
│   │   └── productRoutes.js        # Definición de rutas
│   ├── services
│   │   └── productService.js     # Lógica de negocio
│   ├── app.js                    # Configuración de la aplicación
│   └── server.js                 # Punto de entrada del servidor
├── .env                          # Variables de entorno
├── package.json                  # Dependencias del proyecto
└── README.md                     # Documentación
```

- **src:** Contiene todo el código fuente del proyecto.
- **config:** Almacena configuraciones como base de datos o autenticación.
- **controllers:** Gestiona las solicitudes y respuestas HTTP.
- **models:** Representa las entidades o modelos de datos.
- **routes:** Define las rutas disponibles en la API.
- **services:** Contiene la lógica de negocio o el manejo de la base de datos.
- **app.js:** Archivo que configura y aplica middleware, como Express, Morgan, etc.
- **server.js:** Punto de entrada donde se ejecuta el servidor.

## 5. Pasos para crear el CRUD con PostgreSQL
Instalar dependencias necesarias
Primero, instala las dependencias adicionales necesarias para conectarte a PostgreSQL:

```bash
npm install pg pg-hstore sequelize
```

- **pg:** El cliente para conectarse a PostgreSQL.
- **sequelize:** Un ORM para trabajar con bases de datos relacionales como PostgreSQL.

## Creación de los archivos CRUD

### a. **Configuración de Sequelize**
Crea un archivo para la configuración de Sequelize. 
**config/database.js**


```bash
const { Sequelize } = require('sequelize');
require('dotenv').config();

const sequelize = new Sequelize(process.env.DB_NAME, process.env.DB_USER, process.env.DB_PASSWORD, {
  host: process.env.DB_HOST,
  dialect: 'postgres',
  logging: false,
});

sequelize.authenticate()
  .then(() => console.log('Conectado a PostgreSQL'))
  .catch(err => console.error('No se pudo conectar a la base de datos:', err));

module.exports = sequelize;
```

### b. Actualiza el archivo .env con las credenciales de PostgreSQL
**.env**

```bash
DB_NAME=mi_base_de_datos
DB_USER=mi_usuario
DB_PASSWORD=mi_contraseña
DB_HOST=localhost
PORT=4000
```

### c. **Creación del modelo de productos**
Crea un archivo para definir el modelo de productos usando Sequelize. 
productModel.js (dentro de la carpeta models)
Este archivo define el modelo de datos para el producto.

```bash
const { DataTypes } = require('sequelize');
const sequelize = require('../config/database');

const Product = sequelize.define('Product', {
  name: {
    type: DataTypes.STRING,
    allowNull: false,
  },
  price: {
    type: DataTypes.FLOAT,
    allowNull: false,
  },
  description: {
    type: DataTypes.STRING,
    allowNull: false,
  }
}, {
  timestamps: true
});

module.exports = Product;
```

### e. Creacion del CRUD
Ahora, actualiza los servicios y controladores para que usen PostgreSQL a través de Sequelize.
**productService.js**


```bash
const Product = require('../models/productModel');

// Obtener todos los productos
const getAllProducts = async () => {
  return await Product.findAll();
};

// Obtener un producto por ID
const getProductById = async (id) => {
  return await Product.findByPk(id);
};

// Crear un nuevo producto
const createProduct = async (data) => {
  return await Product.create(data);
};

// Actualizar un producto existente
const updateProduct = async (id, data) => {
  const product = await Product.findByPk(id);
  if (product) {
    return await product.update(data);
  }
  return null;
};

// Eliminar un producto
const deleteProduct = async (id) => {
  const product = await Product.findByPk(id);
  if (product) {
    return await product.destroy();
  }
  return null;
};

module.exports = {
  getAllProducts,
  getProductById,
  createProduct,
  updateProduct,
  deleteProduct
};
```

### f. **productController.js** (dentro de la carpeta controllers)
Este archivo contiene las funciones controladoras que se conectan con los servicios.


```bash
const productService = require('../services/productService');

// Obtener todos los productos
const getAllProducts = async (req, res) => {
  try {
    const products = await productService.getAllProducts();
    res.json(products);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

// Obtener un producto por ID
const getProductById = async (req, res) => {
  try {
    const product = await productService.getProductById(req.params.id);
    if (!product) {
      return res.status(404).json({ message: 'Producto no encontrado' });
    }
    res.json(product);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

// Crear un nuevo producto
const createProduct = async (req, res) => {
  try {
    const product = await productService.createProduct(req.body);
    res.status(201).json(product);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

// Actualizar un producto existente
const updateProduct = async (req, res) => {
  try {
    const product = await productService.updateProduct(req.params.id, req.body);
    if (!product) {
      return res.status(404).json({ message: 'Producto no encontrado' });
    }
    res.json(product);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
};

// Eliminar un producto
const deleteProduct = async (req, res) => {
  try {
    const product = await productService.deleteProduct(req.params.id);
    if (!product) {
      return res.status(404).json({ message: 'Producto no encontrado' });
    }
    res.json({ message: 'Producto eliminado' });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
};

module.exports = {
  getAllProducts,
  getProductById,
  createProduct,
  updateProduct,
  deleteProduct
};
```

### g. productRoutes.js (dentro de la carpeta routes)
Este archivo define las rutas para los endpoints CRUD.

```bash
const express = require('express');
const {
  getAllProducts,
  getProductById,
  createProduct,
  updateProduct,
  deleteProduct
} = require('../controllers/productController');

const router = express.Router();

// Ruta para obtener todos los productos
router.get('/', getAllProducts);

// Ruta para obtener un producto por ID
router.get('/:id', getProductById);

// Ruta para crear un nuevo producto
router.post('/', createProduct);

// Ruta para actualizar un producto
router.put('/:id', updateProduct);

// Ruta para eliminar un producto
router.delete('/:id', deleteProduct);

module.exports = router;
```

### h. **app.js** (actualización)
No olvides sincronizar el modelo con la base de datos. Actualiza el archivo app.js para incluir esto:

```bash
const express = require('express');
const morgan = require('morgan');
const cors = require('cors');
require('dotenv').config();
const sequelize = require('./config/database');  // Nueva línea
const productRoutes = require('./routes/productRoutes');

const app = express();

app.use(morgan('dev'));
app.use(cors());
app.use(express.json());

// Sincronizar base de datos
sequelize.sync().then(() => {
  console.log('Base de datos sincronizada');
}).catch(error => console.log('Error al sincronizar la base de datos:', error));

// Rutas
app.use('/api/products', productRoutes);

module.exports = app;
```

### i. Ejecutar el proyecto
Una vez que hayas completado todo, sigue estos pasos para ejecutar el proyecto:

1. Iniciar PostgreSQL: Asegúrate de tener PostgreSQL corriendo y la base de datos creada.
2. Iniciar el servidor: Usa el siguiente comando para iniciar tu aplicación.

```bash
npm start
```

### Hacer las pruebas en postman 

Endpoints del CRUD de Productos
Obtener todos los productos

Método: GET
```bash
URL: http://localhost:4000/api/products
```
Descripción: Obtiene la lista de todos los productos.
Respuesta esperada (ejemplo):
json
```bash
[
  {
    "id": 1,
    "name": "Producto A",
    "price": 100.0,
    "description": "Descripción del Producto A",
  },
  {
    "id": 2,
    "name": "Producto B",
    "price": 200.0,
    "description": "Descripción del Producto B",
  }
]
```
Obtener un producto por ID

Método: GET
```bash
URL: http://localhost:4000/api/products/:id
```
Descripción: Obtiene un producto específico por su ID.
Parámetros: :id (ID del producto)
Respuesta esperada (ejemplo):
json
```bash
{
  "id": 1,
  "name": "Producto A",
  "price": 100.0,
  "description": "Descripción del Producto A",

}
```
Crear un nuevo producto

Método: POST
```bash
URL: http://localhost:4000/api/products
```
Descripción: Crea un nuevo producto.
Cuerpo de la solicitud (JSON):
json
```bash
{
  "name": "Producto C",
  "price": 300.0,
  "description": "Descripción del Producto C"
}
```
Respuesta esperada (ejemplo):
json
```bash
{
  "id": 3,
  "name": "Producto C",
  "price": 300.0,
  "description": "Descripción del Producto C",
}
```
Actualizar un producto existente

Método: PUT
```bash
URL: http://localhost:4000/api/products/:id
```
Descripción: Actualiza un producto existente.
Parámetros: :id (ID del producto)
Cuerpo de la solicitud (JSON):
json
```bash
{
  "name": "Producto Actualizado",
  "price": 150.0,
  "description": "Descripción del Producto Actualizado"
}
```
Respuesta esperada (ejemplo):
json
```bash
{
  "id": 1,
  "name": "Producto Actualizado",
  "price": 150.0,
  "description": "Descripción del Producto Actualizado",

}
```
Eliminar un producto

Método: DELETE
```bash
URL: http://localhost:4000/api/products/:id
```
Descripción: Elimina un producto existente por su ID.
Parámetros: :id (ID del producto)
Respuesta esperada (ejemplo):
json
```bash
{
  "message": "Producto eliminado"
}
```

