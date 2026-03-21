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

