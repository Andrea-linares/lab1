<template>
  <div class="container">
    <header>
      <h1> Sistema de Gestión de Inventario</h1>
      <p>Controla tus productos de manera eficiente</p>
    </header>

    <!-- Sección de formulario para agregar productos -->
    <div class="form-section">
      <h2> Agregar Nuevo Producto</h2>
      <div class="form-group">
        <label for="nombre">Nombre del Producto *</label>
        <input type="text" id="nombre" v-model="nuevoProducto.nombre" placeholder="Ej: Arroz 1kg">
      </div>

      <div class="form-group">
        <label for="categoria">Categoría *</label>
        <select id="categoria" v-model="nuevoProducto.categoria">
          <option value="">Seleccionar categoría</option>
          <option value="Alimentos">Alimentos</option>
          <option value="Bebidas">Bebidas</option>
          <option value="Limpieza">Limpieza</option>
          <option value="Electrónica">Electrónica</option>
          <option value="Papelería">Papelería</option>
        </select>
      </div>

      <div class="form-row">
        <div class="form-group">
          <label for="precio">Precio (Q) *</label>
          <input type="number" id="precio" v-model="nuevoProducto.precio" placeholder="0.00" step="0.01">
        </div>

        <div class="form-group">
          <label for="cantidad">Cantidad en Stock *</label>
          <input type="number" id="cantidad" v-model="nuevoProducto.cantidad" placeholder="0">
        </div>
      </div>

      <div class="form-actions">
        <button @click="agregarProducto" :disabled="!formularioValido" class="btn-primary">
          Agregar Producto
        </button>
        <button @click="limpiarFormulario" class="btn-secondary">
          Limpiar
        </button>
      </div>

      <!-- Mensaje de validación -->
      <p v-if="mensajeValidacion" class="error-message">{{ mensajeValidacion }}</p>
    </div>

    <!-- Sección de filtros y estadísticas -->
    <div class="filters-section">
      <div class="filter-group">
        <label for="filtroCategoria">Filtrar por categoría:</label>
        <select id="filtroCategoria" v-model="filtroCategoria">
          <option value="">Todas las categorías</option>
          <option value="Alimentos">Alimentos</option>
          <option value="Bebidas">Bebidas</option>
          <option value="Limpieza">Limpieza</option>
          <option value="Electrónica">Electrónica</option>
          <option value="Papelería">Papelería</option>
        </select>
      </div>

      <div class="stats-cards">
        <div class="stat-card">
          <h3>Total Productos</h3>
          <p class="stat-number">{{ productosFiltrados.length }}</p>
        </div>
        <div class="stat-card">
          <h3>Valor Total Stock</h3>
          <p class="stat-number">Q{{ valorTotalStock.toFixed(2) }}</p>
        </div>
        <div class="stat-card">
          <h3>Productos Bajos Stock</h3>
          <p class="stat-number" :class="{ 'warning': productosBajoStock > 0 }">
            {{ productosBajoStock }}
          </p>
        </div>
      </div>
    </div>

    <!-- Tabla de productos -->
    <div class="table-section">
      <h2> Lista de Productos</h2>
      <div v-if="productosFiltrados.length === 0" class="empty-state">
        <p>No hay productos registrados.</p>
        <p>¡Agrega tu primer producto!</p>
      </div>

      <table v-else class="product-table">
        <thead>
          <tr>
            <th>#</th>
            <th>Producto</th>
            <th>Categoría</th>
            <th>Precio (Q)</th>
            <th>Stock</th>
            <th>Valor Total</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(producto, index) in productosFiltrados" :key="producto.id" :class="{ 'low-stock': producto.cantidad < 5 }">
            <td>{{ index + 1 }}</td>
            <td>{{ producto.nombre }}</td>
            <td><span class="category-badge">{{ producto.categoria }}</span></td>
            <td>Q{{ producto.precio.toFixed(2) }}</td>
            <td :class="{ 'stock-warning': producto.cantidad < 5 }">
              {{ producto.cantidad }}
              <span v-if="producto.cantidad < 5" class="warning-icon"></span>
            </td>
            <td>Q{{ (producto.precio * producto.cantidad).toFixed(2) }}</td>
            <td>
              <button @click="eliminarProducto(producto.id)" class="btn-delete" title="Eliminar producto">
                
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Sección de productos con bajo stock (usando v-if) -->
    <div v-if="productosBajoStock > 0" class="alert-section">
      <div class="alert-warning">
        <strong> Atención:</strong> Hay {{ productosBajoStock }} producto(s) con stock bajo (menos de 5 unidades).
        <span v-for="producto in productosConBajoStock" :key="producto.id" class="alert-product">
          {{ producto.nombre }} ({{ producto.cantidad }})
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

// Variables reactivas
const nuevoProducto = ref({
  nombre: '',
  categoria: '',
  precio: '',
  cantidad: ''
})

const productos = ref([])
const filtroCategoria = ref('')
const mensajeValidacion = ref('')

// ID autoincremental
let nextId = 1

// Computed: productos filtrados por categoría
const productosFiltrados = computed(() => {
  if (filtroCategoria.value === '') {
    return productos.value
  }
  return productos.value.filter(p => p.categoria === filtroCategoria.value)
})

// Computed: valor total del stock
const valorTotalStock = computed(() => {
  return productos.value.reduce((total, producto) => {
    return total + (producto.precio * producto.cantidad)
  }, 0)
})

// Computed: cantidad de productos con bajo stock
const productosBajoStock = computed(() => {
  return productos.value.filter(p => p.cantidad < 5).length
})

// Computed: lista de productos con bajo stock
const productosConBajoStock = computed(() => {
  return productos.value.filter(p => p.cantidad < 5)
})

// Computed: validación del formulario
const formularioValido = computed(() => {
  return nuevoProducto.value.nombre.trim() !== '' &&
         nuevoProducto.value.categoria !== '' &&
         nuevoProducto.value.precio !== '' &&
         nuevoProducto.value.cantidad !== '' &&
         parseFloat(nuevoProducto.value.precio) > 0 &&
         parseInt(nuevoProducto.value.cantidad) >= 0
})

// Función para validar campos específicos
const validarCampos = () => {
  if (nuevoProducto.value.nombre.trim() === '') {
    mensajeValidacion.value = ' El nombre del producto es obligatorio'
    return false
  }
  if (nuevoProducto.value.categoria === '') {
    mensajeValidacion.value = ' Debes seleccionar una categoría'
    return false
  }
  if (nuevoProducto.value.precio === '' || parseFloat(nuevoProducto.value.precio) <= 0) {
    mensajeValidacion.value = ' El precio debe ser mayor a 0'
    return false
  }
  if (nuevoProducto.value.cantidad === '' || parseInt(nuevoProducto.value.cantidad) < 0) {
    mensajeValidacion.value = ' La cantidad no puede ser negativa'
    return false
  }
  mensajeValidacion.value = ''
  return true
}

// Función para agregar producto
const agregarProducto = () => {
  if (!validarCampos()) {
    return
  }
  
  const producto = {
    id: nextId++,
    nombre: nuevoProducto.value.nombre.trim(),
    categoria: nuevoProducto.value.categoria,
    precio: parseFloat(nuevoProducto.value.precio),
    cantidad: parseInt(nuevoProducto.value.cantidad)
  }
  
  productos.value.push(producto)
  limpiarFormulario()
  mensajeValidacion.value = ' Producto agregado exitosamente'
  
  // Limpiar mensaje después de 3 segundos
  setTimeout(() => {
    if (mensajeValidacion.value === ' Producto agregado exitosamente') {
      mensajeValidacion.value = ''
    }
  }, 3000)
}

// Función para limpiar formulario
const limpiarFormulario = () => {
  nuevoProducto.value = {
    nombre: '',
    categoria: '',
    precio: '',
    cantidad: ''
  }
  mensajeValidacion.value = ''
}

// Función para eliminar producto
const eliminarProducto = (id) => {
  const index = productos.value.findIndex(p => p.id === id)
  if (index !== -1) {
    productos.value.splice(index, 1)
    mensajeValidacion.value = ' Producto eliminado'
    setTimeout(() => {
      if (mensajeValidacion.value === ' Producto eliminado') {
        mensajeValidacion.value = ''
      }
    }, 2000)
  }
}
</script>


<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  background: white;
  border-radius: 20px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.3);
  overflow: hidden;
  padding: 30px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

body {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  padding: 20px;
  margin: 0;
}

header {
  text-align: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 2px solid #e0e0e0;
}

header h1 {
  color: #333;
  font-size: 2rem;
  margin-bottom: 10px;
}

header p {
  color: #666;
}

/* Formulario */
.form-section {
  background: #f8f9fa;
  padding: 25px;
  border-radius: 15px;
  margin-bottom: 30px;
}

.form-section h2 {
  margin-bottom: 20px;
  color: #333;
  font-size: 1.3rem;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 600;
  color: #555;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 10px 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

.form-group input:focus,
.form-group select:focus {
  outline: none;
  border-color: #667eea;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.btn-primary,
.btn-secondary {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102,126,234,0.4);
}

.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-secondary {
  background: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background: #5a6268;
}

.error-message {
  color: #dc3545;
  margin-top: 10px;
  font-size: 0.9rem;
}

/* Filtros y estadísticas */
.filters-section {
  margin-bottom: 30px;
}

.filter-group {
  margin-bottom: 20px;
}

.filter-group select {
  padding: 8px 15px;
  border-radius: 8px;
  border: 1px solid #ddd;
  margin-left: 10px;
}

.stats-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

.stat-card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
  border-radius: 15px;
  text-align: center;
}

.stat-card h3 {
  font-size: 0.9rem;
  margin-bottom: 10px;
  opacity: 0.9;
}

.stat-number {
  font-size: 2rem;
  font-weight: bold;
}

.stat-number.warning {
  color: #ffc107;
}

/* Tabla */
.table-section {
  margin-top: 20px;
}

.table-section h2 {
  margin-bottom: 15px;
  color: #333;
}

.empty-state {
  text-align: center;
  padding: 50px;
  background: #f8f9fa;
  border-radius: 10px;
  color: #666;
}

.product-table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.product-table th,
.product-table td {
  padding: 12px 15px;
  text-align: left;
  border-bottom: 1px solid #e0e0e0;
}

.product-table th {
  background: #f8f9fa;
  font-weight: 600;
  color: #555;
}

.product-table tr:hover {
  background: #f8f9fa;
}

.product-table tr.low-stock {
  background: #fff3cd;
}

.category-badge {
  background: #e0e0e0;
  padding: 4px 8px;
  border-radius: 5px;
  font-size: 0.85rem;
}

.stock-warning {
  color: #dc3545;
  font-weight: bold;
}

.warning-icon {
  margin-left: 5px;
}

.btn-delete {
  background: #dc3545;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-delete:hover {
  background: #c82333;
}

/* Alerta */
.alert-section {
  margin-top: 20px;
}

.alert-warning {
  background: #fff3cd;
  border-left: 4px solid #ffc107;
  padding: 15px;
  border-radius: 8px;
  color: #856404;
}

.alert-product {
  display: inline-block;
  background: #ffeeba;
  padding: 3px 8px;
  border-radius: 5px;
  margin: 5px;
  font-size: 0.85rem;
}

/* Responsive */
@media (max-width: 768px) {
  .container {
    padding: 15px;
  }
  
  .form-row {
    grid-template-columns: 1fr;
  }
  
  .product-table {
    font-size: 0.85rem;
  }
  
  .product-table th,
  .product-table td {
    padding: 8px 10px;
  }
  
  .stats-cards {
    grid-template-columns: 1fr;
  }
}
</style>
