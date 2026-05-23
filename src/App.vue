<template>
  <div class="container">
    <div class="header">
      <h1>🍕 Restaurante Delicioso</h1>
      <p>Selecciona tus platos favoritos</p>
    </div>

    <div class="tabs">
      <button 
        class="tab-btn" 
        v-on:click="currentView = 'menu'"
        :class="{ active: currentView === 'menu' }"
      >
        📋 Menú
      </button>

    </div>

    <button 
      class="fab" 
      v-on:click="currentView = 'addProduct'"
      title="Agregar Producto"
      v-show="currentView === 'menu'"
    >
      ➕
    </button>

    <button 
      class="fab fab-cart" 
      v-on:click="currentView = 'cart'"
      :class="{ active: currentView === 'cart' }"
      title="Carrito"
    >
      🛒 {{ cart.length }}
    </button>

    <div v-if="currentView === 'menu'">       
      <div class="menu-grid">

        <div v-for="product in products" :key="product.id" class="product-card"> 
          <img :src="product.image" :alt="product.name">
          <div class="product-info">
            <h3>{{ product.name }}</h3>
            <div class="price">{{ product.price }} COP</div>
            <div>Disponible: {{ product.available }}</div>
            <button class="add-btn" v-on:click="agregarAlCarrito(product.id)" :disabled="product.available <= 0">
              Añadir al carrito
            </button>
            <button class="delete-btn" v-on:click="eliminarProducto(product.id)">
              🗑️ Eliminar
            </button>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="currentView === 'addProduct'">
      <div class="add-product-form">
        <h2>➕ Agregar Nuevo Producto</h2>
        <div class="form-group">
          <label>Nombre del producto</label>
          <input v-model="newProduct.name" type="text" placeholder="Ej: 🍔 Hamburguesa Especial">
        </div>
        <div class="form-group">
          <label>Precio (COP)</label>
          <input v-model.number="newProduct.price" type="number" placeholder="Ej: 45000">
        </div>
        <div class="form-group">
          <label>URL de la imagen</label>
          <input v-model="newProduct.image" type="text" placeholder="https://...">
        </div>
        <div class="form-group">
          <label>Cantidad disponible</label>
          <input v-model.number="newProduct.available" type="number" placeholder="Ej: 50">
        </div>
        <button class="add-btn" v-on:click="guardarProducto">Guardar Producto</button>
      </div>
    </div>
    

    <div v-else-if="currentView === 'cart'">
      <div v-if="cart.length === 0" class="empty-cart">
        <h2>🛒 Tu carrito está vacío</h2>
        <p>¡Empieza a añadir productos desde el menú!</p>
        <button class="tab-btn" v-on:click="currentView = 'menu'">Ver Menú</button>
      </div>



      <div v-else class="cart-list">
        <div v-for="item in cart" :key="item.id" class="cart-item">
          <div class="cart-details">
            <h4>{{ item.name }}</h4>
            <span class="cart-price">{{ item.price }} COP</span>
          </div>
          <div class="quantity-controls">
            <button class="qty-btn" v-on:click="cambiarCantidad(item.id, -1)">-</button>
            <span>{{ item.quantity }}</span>
            <button class="qty-btn" v-on:click="cambiarCantidad(item.id, 1)">+</button>
          </div>
          <button class="remove-btn" v-on:click="removerDelCarrito(item.id)">
            Remover
          </button>
        </div>
        

        <div class="total">
          <h3>Total: <span class="total-amount">{{ calcularTotal() }} COP</span></h3>
          <div style="margin-top: 20px;">
            <button class="add-btn" style="width: 200px;" v-on:click="vaciarCarrito">
              Vaciar Carrito
            </button>
          </div>
          <div style="margin-top: 15px;">
            <button class="checkout-btn" style="width: 200px;" v-on:click="finalizarPedido">
              ✅ Finalizar Pedido
            </button>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="currentView === 'invoice'" class="invoice-container">
      <div class="invoice" ref="invoiceRef">
        <div class="invoice-header">
          <h2>🧾 Factura</h2>
          <div class="invoice-meta">
            <p><strong>Nº Factura:</strong> #{{ invoiceNumber }}</p>
            <p><strong>Fecha:</strong> {{ invoiceDate }}</p>
            <p><strong>Restaurante Delicioso</strong></p>
          </div>
        </div>

        <div class="invoice-items">
          <div class="invoice-row header">
            <span>Producto</span>
            <span>Cant.</span>
            <span>Precio</span>
            <span>Subtotal</span>
          </div>
          <div v-for="item in cart" :key="item.id" class="invoice-row">
            <span>{{ item.name }}</span>
            <span>{{ item.quantity }}</span>
            <span>{{ item.price.toFixed(2) }} COP</span>
            <span>{{ (item.price * item.quantity).toFixed(2) }} COP</span>
          </div>
        </div>

        <div class="invoice-total">
          <h3>Total a pagar: <span>{{ calcularTotal() }} COP</span></h3>
        </div>

        <div class="invoice-actions">
          <button class="add-btn" v-on:click="nuevoPedido" style="margin-bottom: 10px;">
            🍽️ Nuevo Pedido
          </button>
          <button class="checkout-btn" v-on:click="descargarPDF">
            📄 Descargar PDF
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import jsPDF from 'jspdf'
import html2canvas from 'html2canvas'

// Productos del menú
const products = ref([
  { id: 1, name: '🍔 Hamburguesa Clásica', price: 45000, available: 100, image: 'https://images.unsplash.com/photo-1568901346375-23c9450c58cd?w=300&h=200&fit=crop' },
  { id: 2, name: '🍔 Cheeseburger', price: 48000, available: 80, image: 'https://images.unsplash.com/photo-1571091718767-18b5b1457add?w=300&h=200&fit=crop' },
  { id: 3, name: '🍕 Pizza Margherita', price: 50000, available: 50, image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/c/c8/Pizza_Margherita_stu_spivack.jpg/1280px-Pizza_Margherita_stu_spivack.jpg' },
  { id: 4, name: '🍕 Pizza Pepperoni', price: 55000, available: 40, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSzCw_1kFdC1njIh3bNk8GX5SWK63bWux7rCUAbd5VLv34ZVeeMEfoB0U-tKLY54DXEyOqgkshHjoHC3_DCgMG1yc6RyB3tTdSCb-FHeRh4MLLIjMCqrBh73kVmBmS99w&s=10&ec=121638510' },
  { id: 5, name: '🍝 Pasta Carbonara', price: 49000, available: 60, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTCG_ceFRB7gtNE57WFZ22oSe46hDfHq7dNkDvSDTzFtwxGKVJSzYt_n96oIUxfPEYaW6ohzjZgbzAejccHdtreB-aur7-Qyp1qRia4QOrxgK-ed5oTWsR1hMoU42bGjg&s=10&ec=121638510' },
  { id: 6, name: '🍝 Spaghetti Boloñesa', price: 51000, available: 70, image: 'https://cdn.stoneline.de/media/c5/63/4f/1727429313/spaghetti-bolognese.jpeg?ts=1727429313' },
  { id: 7, name: '🥗 Ensalada César', price: 34000, available: 90, image: 'https://i.etsystatic.com/64700415/r/il/1b9c42/7914965166/il_794xN.7914965166_o5dq.jpg' },
  { id: 8, name: '🥗 Ensalada Mediterránea', price: 38000, available: 85, image: 'https://images.unsplash.com/photo-1546069901-ba9599a7e63c?w=300&h=200&fit=crop' },
  { id: 9, name: '🍗 Pollo Asado', price: 57000, available: 45, image: 'https://www.mycolombianrecipes.com/wp-content/uploads/2009/11/Pollo-Asado1.JPG' },
  { id: 10, name: '🍗 Pollo BBQ', price: 61000, available: 55, image: 'https://popmenucloud.com/cdn-cgi/image/width%3D1200%2Cheight%3D1200%2Cfit%3Dscale-down%2Cformat%3Dauto%2Cquality%3D60/coejiwxh/86e3017d-dfe9-41f1-8c7c-75eb3feb0fab.png' },
  { id: 11, name: '🥩 Filete de Res', price: 80000, available: 30, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQTNHuB89DUT9LRbtUwYdgbQsoFUrUpo1Oh8Q&s' },
  { id: 12, name: '🐟 Salmón a la Plancha', price: 70000, available: 35, image: 'https://images.unsplash.com/photo-1467003909585-2f8a72700288?w=300&h=200&fit=crop' },
  { id: 13, name: '🌮 Tacos Al Pastor', price: 39000, available: 75, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTXiNsnnAfPLpQdWnxXVBCSpen0f_Prp31lig&s' },
  { id: 14, name: '🌯 Burrito', price: 45000, available: 65, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRtTqN58dYomkxVX-npQrq9W8A1YhBnUlrWOg&s' },
  { id: 15, name: '🍛 Curry Vegetariano', price: 46000, available: 50, image: 'https://images.unsplash.com/photo-1540189549336-e6e99c3679fe?w=300&h=200&fit=crop' },
  { id: 16, name: '🍜 Pad Thai', price: 52000, available: 40, image: 'https://inquiringchef.com/wp-content/uploads/2023/02/Authentic-Pad-Thai_square-1908-500x375.jpg' },
  { id: 17, name: '🍟 Papas Fritas', price: 20000, available: 120, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQigYPCWW5JEV2bHWdhRHEX5hWlZ4eHIoOksg&s' },
  { id: 18, name: '🧅 Cebollas Fritas', price: 22000, available: 100, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSNBWAykjm1gavFb9W3jttbWTMMuXmsEqrFQw&s' },
  { id: 19, name: '🍦 Tiramisú', price: 24000, available: 80, image: 'https://kitchen-by-the-sea.com/wp-content/uploads/2022/04/Tiramisu-16.jpg' },
  { id: 20, name: '🍮 Flan', price: 20500, available: 95, image: 'https://images.unsplash.com/photo-1578985545062-69928b1d9587?w=300&h=200&fit=crop' },
  { id: 21, name: '🍩 Cheesecake', price: 26000, available: 70, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRpS6yaJcvttjlTPYohqV_7Kgc-WjBUwY8iYw&s' },
  { id: 22, name: '🍹 Mojito', price: 30000, available: 60, image: 'https://images.unsplash.com/photo-1576092768241-dec231879fc3?w=300&h=200&fit=crop' },
  { id: 23, name: '🍺 Cerveza', price: 17000, available: 150, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTA-usyISLS65O_RplJQ6qyeRmcbodqw7OxTw&s' },
  { id: 24, name: '🍷 Vino Tinto', price: 27000, available: 50, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTd943sxnAjf5BqXapY-YUGTnNdYR45f9cK5A&s' },
  { id: 25, name: '🥤 Refresco', price: 12000, available: 200, image: 'https://images.unsplash.com/photo-1576092768241-dec231879fc3?w=300&h=200&fit=crop' },
  { id: 26, name: '☕ Café Americano', price: 10000, available: 180, image: 'https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?w=300&h=200&fit=crop' },
  { id: 27, name: '🧋 Bubble Tea', price: 19000, available: 90, image: 'https://www.figjar.com/wp-content/uploads/2025/03/strawberry-bubble-tea.jpg' },
  { id: 28, name: '🥛 Batido Fresa', price: 22000, available: 85, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTzJIIeT1-lD6TiLCoTopZvjSXpd2i1Grmc2A&s' },
  { id: 29, name: '🍉 Agua Fresca Sandía', price: 15000, available: 110, image: 'https://lolascocina.com/wp-content/uploads/2015/07/2025.05.17-Agua-de-Sandia-WEB-13.jpg' },
  { id: 30, name: '🧋 Té Helado', price: 13000, available: 160, image: 'https://imag.bonviveur.com/te-helado.jpg' }
])

// Estado del carrito
const cart = ref([])

// Vista actual
const currentView = ref('menu')

// Datos de factura
const invoiceNumber = ref('')
const invoiceDate = ref('')

// Nuevo producto form
const newProduct = ref({
  name: '',
  price: 0,
  available: 0,
  image: ''
})

const guardarProducto = () => {
  if (!newProduct.value.name || !newProduct.value.image || newProduct.value.price <= 0) {
    alert('Por favor completa todos los campos correctamente')
    return
  }

  const newId = Math.max(...products.value.map(p => p.id), 0) + 1

  products.value.push({
    id: newId,
    name: newProduct.value.name,
    price: newProduct.value.price,
    available: newProduct.value.available,
    image: newProduct.value.image
  })

  newProduct.value = { name: '', price: 0, available: 0, image: '' }
  currentView.value = 'menu'
}

const eliminarProducto = (productId) => {
  if (!confirm('¿Estás seguro de que deseas eliminar este producto?')) return
  const index = products.value.findIndex(p => p.id === productId)
  if (index > -1) {
    products.value.splice(index, 1)
  }
}

// Añadir producto al carrito
const agregarAlCarrito = (productId) => {
  const product = products.value.find(p => p.id === productId)
  if (!product || product.available <= 0) return

  const existingItem = cart.value.find(item => item.id === productId)

  if (existingItem) {
    existingItem.quantity += 1
  } else {
    cart.value.push({
      ...product,
      quantity: 1
    })
  }
  product.available -= 1
}

// Cambiar cantidad
const cambiarCantidad = (productId, change) => {
  const item = cart.value.find(item => item.id === productId)
  const product = products.value.find(p => p.id === productId)
  if (!item || !product) return

  if (change > 0) {
    if (product.available > 0) {
      item.quantity += 1
      product.available -= 1
    }
  } else {
    if (item.quantity > 1) {
      item.quantity -= 1
      product.available += 1
    } else {
      removerDelCarrito(productId)
    }
  }
}

// Remover del carrito
const removerDelCarrito = (productId) => {
  const index = cart.value.findIndex(item => item.id === productId)
  if (index > -1) {
    const item = cart.value[index]
    const product = products.value.find(p => p.id === productId)
    if (product) {
      product.available += item.quantity
    }
    cart.value.splice(index, 1)
  }
}

// Vaciar carrito
const vaciarCarrito = () => {
  for (const item of cart.value) {
    const product = products.value.find(p => p.id === item.id)
    if (product) {
      product.available += item.quantity
    }
  }
  cart.value = []
}

// Calcular total
const calcularTotal = () => {
  let total = 0
  for (let i = 0; i < cart.value.length; i++) {
    total += cart.value[i].price * cart.value[i].quantity
  }
  return total.toFixed(2)
}

// Finalizar pedido y generar factura
const finalizarPedido = () => {
  const now = new Date()
  invoiceNumber.value = Math.floor(Math.random() * 1000000).toString().padStart(6, '0')
  invoiceDate.value = now.toLocaleString('es-CO', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
  currentView.value = 'invoice'
}

// Referencia para la factura (PDF)
const invoiceRef = ref(null)

// Descargar factura en PDF
const descargarPDF = async () => {
  if (!invoiceRef.value) return

  try {
    const canvas = await html2canvas(invoiceRef.value, {
      scale: 2,
      backgroundColor: '#ffffff'
    })
    const imgData = canvas.toDataURL('image/png')
    const pdf = new jsPDF('p', 'mm', 'a4')
    const pdfWidth = pdf.internal.pageSize.getWidth()
    const pdfHeight = (canvas.height * pdfWidth) / canvas.width
    pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight)
    pdf.save(`Factura_${invoiceNumber.value}.pdf`)
  } catch (error) {
    alert('Error al generar el PDF')
  }
}

// Nuevo pedido
const nuevoPedido = () => {
  cart.value = []
  currentView.value = 'menu'
}
</script>

<style scoped>
/* Estilos específicos del componente si es necesario */
.container {
  min-height: 100vh;
}
</style>
