<template>
  <div class="container">
    <div class="header">
      <h1>🍕 Restaurante Delicioso</h1>
      <p>Selecciona tus platos favoritos</p>
    </div>

    <div v-if="notification.show" :class="['notification', notification.type]">
      {{ notification.message }}
    </div>

    <div v-if="confirmDialog.show" class="confirm-overlay" v-on:click="cerrarConfirm">
      <div class="confirm-modal" v-on:click.stop>
        <p>{{ confirmDialog.message }}</p>
        <div class="confirm-buttons">
          <button class="tab-btn" v-on:click="confirmDialog.onConfirm(); cerrarConfirm()">Sí</button>
          <button class="tab-btn" v-on:click="cerrarConfirm">Cancelar</button>
        </div>
      </div>
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

    <div v-if="showCategoryMenu" class="dropdown-overlay" v-on:click="showCategoryMenu = false"></div>

    <div class="fab-bar" v-show="currentView === 'menu'">
      <button 
        class="fab-btn" 
        v-on:click="currentView = 'addProduct'"
        title="Agregar Producto"
        v-show="currentView === 'menu'"
      >
        ➕
      </button>

      <div class="fab-category-section">
        <button 
          class="fab-btn" 
          v-on:click="showCategoryMenu = !showCategoryMenu"
          title="Filtrar por categoría"
        >
          📂
        </button>
        <div v-if="showCategoryMenu" class="category-dropdown">
          <button 
            v-for="cat in categories" 
            :key="cat.key"
            class="category-option"
            :class="{ active: selectedCategory === cat.key }"
            v-on:click="selectedCategory = cat.key; showCategoryMenu = false"
          >
            {{ cat.label }}
          </button>
        </div>
      </div>

      <button 
        class="fab-btn fab-cart-btn" 
        v-on:click="currentView = 'cart'"
        :class="{ active: currentView === 'cart' }"
        title="Carrito"
      >
        🛒 {{ cart.length }}
      </button>
    </div>

    <div v-if="currentView === 'menu'">
      <div class="category-filter">
        <select v-model="selectedCategory" class="category-select">
          <option v-for="cat in categories" :key="cat.key" :value="cat.key">{{ cat.label }}</option>
        </select>
      </div>

      <div class="menu-grid">
        <div v-for="product in filteredProducts" :key="product.id" class="product-card"> 
          <img :src="product.image" :alt="product.name">
          <div class="product-info">
            <h3>{{ product.name }}</h3>
            <div class="price">{{ formatoPrecio(product.price) }}</div>
            <div>Disponible: {{ product.available }}</div>
            <button class="add-btn" v-on:click="agregarAlCarrito(product.id)" :disabled="product.available <= 0">
              Añadir al carrito
            </button>
            <button class="delete-btn" v-on:click="confirmarEliminar(product.id)">
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
          <label>Categoría</label>
          <select v-model="newProduct.category" class="search-input">
            <option v-for="cat in categories" :key="cat.key" :value="cat.key">{{ cat.label }}</option>
          </select>
        </div>
        <div class="form-group">
          <label>Precio (COP)</label>
          <input v-model.number="newProduct.price" type="number" placeholder="Ej: 45000" min="0">
        </div>
        <div class="form-group">
          <label>URL de la imagen</label>
          <input v-model="newProduct.image" type="text" placeholder="https://...">
        </div>
        <div class="form-group">
          <label>Cantidad disponible</label>
          <input v-model.number="newProduct.available" type="number" placeholder="Ej: 50" min="0">
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
            <span class="cart-price">{{ formatoPrecio(item.price) }}</span>
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
          <h3>Total: <span class="total-amount">{{ formatoPrecio(calcularTotal()) }}</span></h3>
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
            <span>{{ formatoPrecio(item.price) }}</span>
            <span>{{ formatoPrecio(item.price * item.quantity) }}</span>
          </div>
        </div>

        <div class="invoice-total">
          <h3>Total a pagar: <span>{{ formatoPrecio(calcularTotal()) }}</span></h3>
        </div>
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
</template>

<script setup>
import { ref, computed } from 'vue'
import jsPDF from 'jspdf'
import html2canvas from 'html2canvas'

const formatoPrecio = (value) => {
  return '$' + Math.round(value).toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.') + ' COP'
}

const notificar = (message, type = 'success') => {
  notification.value = { show: true, message, type }
  setTimeout(() => {
    notification.value.show = false
  }, 3000)
}

const cerrarConfirm = () => {
  confirmDialog.value.show = false
}

const categories = [
  { key: 'todas', label: '🍽️ Todas' },
  { key: 'hamburguesas', label: '🍔 Hamburguesas' },
  { key: 'pizzas', label: '🍕 Pizzas' },
  { key: 'pastas', label: '🍝 Pastas' },
  { key: 'ensaladas', label: '🥗 Ensaladas' },
  { key: 'pollo', label: '🍗 Pollo' },
  { key: 'carnes', label: '🥩 Carnes' },
  { key: 'mexicanos', label: '🌮 Mexicanos' },
  { key: 'acompanamientos', label: '🍟 Acompañamientos' },
  { key: 'postres', label: '🍰 Postres' },
  { key: 'bebidas', label: '🥤 Bebidas' }
]

const selectedCategory = ref('todas')
const showCategoryMenu = ref(false)

const products = ref([
  { id: 1, name: '🍔 Hamburguesa Clásica', category: 'hamburguesas', price: 45000, available: 100, image: 'https://images.unsplash.com/photo-1568901346375-23c9450c58cd?w=300&h=200&fit=crop' },
  { id: 2, name: '🍔 Cheeseburger', category: 'hamburguesas', price: 48000, available: 80, image: 'https://images.unsplash.com/photo-1571091718767-18b5b1457add?w=300&h=200&fit=crop' },
  { id: 3, name: '🍕 Pizza Margherita', category: 'pizzas', price: 50000, available: 50, image: 'https://upload.wikimedia.org/wikipedia/commons/thumb/c/c8/Pizza_Margherita_stu_spivack.jpg/1280px-Pizza_Margherita_stu_spivack.jpg' },
  { id: 4, name: '🍕 Pizza Pepperoni', category: 'pizzas', price: 55000, available: 40, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSzCw_1kFdC1njIh3bNk8GX5SWK63bWux7rCUAbd5VLv34ZVeeMEfoB0U-tKLY54DXEyOqgkshHjoHC3_DCgMG1yc6RyB3tTdSCb-FHeRh4MLLIjMCqrBh73kVmBmS99w&s=10&ec=121638510' },
  { id: 5, name: '🍝 Pasta Carbonara', category: 'pastas', price: 49000, available: 60, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTCG_ceFRB7gtNE57WFZ22oSe46hDfHq7dNkDvSDTzFtwxGKVJSzYt_n96oIUxfPEYaW6ohzjZgbzAejccHdtreB-aur7-Qyp1qRia4QOrxgK-ed5oTWsR1hMoU42bGjg&s=10&ec=121638510' },
  { id: 6, name: '🍝 Spaghetti Boloñesa', category: 'pastas', price: 51000, available: 70, image: 'https://cdn.stoneline.de/media/c5/63/4f/1727429313/spaghetti-bolognese.jpeg?ts=1727429313' },
  { id: 7, name: '🥗 Ensalada César', category: 'ensaladas', price: 34000, available: 90, image: 'https://i.etsystatic.com/64700415/r/il/1b9c42/7914965166/il_794xN.7914965166_o5dq.jpg' },
  { id: 8, name: '🥗 Ensalada Mediterránea', category: 'ensaladas', price: 38000, available: 85, image: 'https://images.unsplash.com/photo-1546069901-ba9599a7e63c?w=300&h=200&fit=crop' },
  { id: 9, name: '🍗 Pollo Asado', category: 'pollo', price: 57000, available: 45, image: 'https://www.mycolombianrecipes.com/wp-content/uploads/2009/11/Pollo-Asado1.JPG' },
  { id: 10, name: '🍗 Pollo BBQ', category: 'pollo', price: 61000, available: 55, image: 'https://popmenucloud.com/cdn-cgi/image/width%3D1200%2Cheight%3D1200%2Cfit%3Dscale-down%2Cformat%3Dauto%2Cquality%3D60/coejiwxh/86e3017d-dfe9-41f1-8c7c-75eb3feb0fab.png' },
  { id: 11, name: '🥩 Filete de Res', category: 'carnes', price: 80000, available: 30, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQTNHuB89DUT9LRbtUwYdgbQsoFUrUpo1Oh8Q&s' },
  { id: 12, name: '🐟 Salmón a la Plancha', category: 'carnes', price: 70000, available: 35, image: 'https://images.unsplash.com/photo-1467003909585-2f8a72700288?w=300&h=200&fit=crop' },
  { id: 13, name: '🌮 Tacos Al Pastor', category: 'mexicanos', price: 39000, available: 75, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTXiNsnnAfPLpQdWnxXVBCSpen0f_Prp31lig&s' },
  { id: 14, name: '🌯 Burrito', category: 'mexicanos', price: 45000, available: 65, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRtTqN58dYomkxVX-npQrq9W8A1YhBnUlrWOg&s' },
  { id: 15, name: '🍛 Curry Vegetariano', category: 'acompanamientos', price: 46000, available: 50, image: 'https://images.unsplash.com/photo-1540189549336-e6e99c3679fe?w=300&h=200&fit=crop' },
  { id: 16, name: '🍜 Pad Thai', category: 'acompanamientos', price: 52000, available: 40, image: 'https://inquiringchef.com/wp-content/uploads/2023/02/Authentic-Pad-Thai_square-1908-500x375.jpg' },
  { id: 17, name: '🍟 Papas Fritas', category: 'acompanamientos', price: 20000, available: 120, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQigYPCWW5JEV2bHWdhRHEX5hWlZ4eHIoOksg&s' },
  { id: 18, name: '🧅 Cebollas Fritas', category: 'acompanamientos', price: 22000, available: 100, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSNBWAykjm1gavFb9W3jttbWTMMuXmsEqrFQw&s' },
  { id: 19, name: '🍦 Tiramisú', category: 'postres', price: 24000, available: 80, image: 'https://kitchen-by-the-sea.com/wp-content/uploads/2022/04/Tiramisu-16.jpg' },
  { id: 20, name: '🍮 Flan', category: 'postres', price: 20500, available: 95, image: 'https://images.unsplash.com/photo-1578985545062-69928b1d9587?w=300&h=200&fit=crop' },
  { id: 21, name: '🍩 Cheesecake', category: 'postres', price: 26000, available: 70, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRpS6yaJcvttjlTPYohqV_7Kgc-WjBUwY8iYw&s' },
  { id: 22, name: '🍹 Mojito', category: 'bebidas', price: 30000, available: 60, image: 'https://images.unsplash.com/photo-1576092768241-dec231879fc3?w=300&h=200&fit=crop' },
  { id: 23, name: '🍺 Cerveza', category: 'bebidas', price: 17000, available: 150, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTA-usyISLS65O_RplJQ6qyeRmcbodqw7OxTw&s' },
  { id: 24, name: '🍷 Vino Tinto', category: 'bebidas', price: 27000, available: 50, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTd943sxnAjf5BqXapY-YUGTnNdYR45f9cK5A&s' },
  { id: 25, name: '🥤 Refresco', category: 'bebidas', price: 12000, available: 200, image: 'https://images.unsplash.com/photo-1576092768241-dec231879fc3?w=300&h=200&fit=crop' },
  { id: 26, name: '☕ Café Americano', category: 'bebidas', price: 10000, available: 180, image: 'https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?w=300&h=200&fit=crop' },
  { id: 27, name: '🧋 Bubble Tea', category: 'bebidas', price: 19000, available: 90, image: 'https://www.figjar.com/wp-content/uploads/2025/03/strawberry-bubble-tea.jpg' },
  { id: 28, name: '🥛 Batido Fresa', category: 'bebidas', price: 22000, available: 85, image: 'https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTzJIIeT1-lD6TiLCoTopZvjSXpd2i1Grmc2A&s' },
  { id: 29, name: '🍉 Agua Fresca Sandía', category: 'bebidas', price: 15000, available: 110, image: 'https://lolascocina.com/wp-content/uploads/2015/07/2025.05.17-Agua-de-Sandia-WEB-13.jpg' },
  { id: 30, name: '🧋 Té Helado', category: 'bebidas', price: 13000, available: 160, image: 'https://imag.bonviveur.com/te-helado.jpg' }
])

const filteredProducts = computed(() => {
  if (selectedCategory.value === 'todas') return products.value
  return products.value.filter(p => p.category === selectedCategory.value)
})

const cart = ref([])
const currentView = ref('menu')

const invoiceNumber = ref('')
const invoiceDate = ref('')

const notification = ref({ show: false, message: '', type: 'success' })

const confirmDialog = ref({ show: false, message: '', onConfirm: () => {} })

const newProduct = ref({
  name: '',
  category: 'hamburguesas',
  price: 0,
  available: 0,
  image: ''
})

const guardarProducto = () => {
  const name = newProduct.value.name.trim()
  const image = newProduct.value.image.trim()
  const price = newProduct.value.price
  const available = newProduct.value.available

  if (!name) {
    notificar('El nombre del producto es obligatorio', 'error')
    return
  }
  if (!image) {
    notificar('La URL de la imagen es obligatoria', 'error')
    return
  }
  if (!/^https?:\/\/.+\.(jpg|jpeg|png|gif|webp|svg|avif|bmp)(\?.*)?$/i.test(image)) {
    notificar('La URL debe ser una imagen válida (jpg, png, gif, webp, etc.)', 'error')
    return
  }
  if (price <= 0) {
    notificar('El precio debe ser mayor a 0', 'error')
    return
  }
  if (available < 0) {
    notificar('La cantidad disponible no puede ser negativa', 'error')
    return
  }

  const newId = Math.max(...products.value.map(p => p.id), 0) + 1

  products.value.push({
    id: newId,
    name,
    category: newProduct.value.category,
    price,
    available,
    image
  })

  newProduct.value = { name: '', category: 'hamburguesas', price: 0, available: 0, image: '' }
  currentView.value = 'menu'
  notificar('Producto guardado exitosamente', 'success')
}

let pendingDeleteId = null

const confirmarEliminar = (productId) => {
  const product = products.value.find(p => p.id === productId)
  if (!product) return
  pendingDeleteId = productId
  confirmDialog.value = {
    show: true,
    message: `¿Estás seguro de eliminar "${product.name}"?`,
    onConfirm: () => {
      const index = products.value.findIndex(p => p.id === pendingDeleteId)
      if (index > -1) {
        products.value.splice(index, 1)
        notificar('Producto eliminado', 'success')
      }
    }
  }
}

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

const vaciarCarrito = () => {
  if (cart.value.length === 0) return
  confirmDialog.value = {
    show: true,
    message: '¿Estás seguro de vaciar el carrito?',
    onConfirm: () => {
      for (const item of cart.value) {
        const product = products.value.find(p => p.id === item.id)
        if (product) {
          product.available += item.quantity
        }
      }
      cart.value = []
      notificar('Carrito vaciado', 'success')
    }
  }
}

const calcularTotal = () => {
  let total = 0
  for (let i = 0; i < cart.value.length; i++) {
    total += cart.value[i].price * cart.value[i].quantity
  }
  return total
}

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

const invoiceRef = ref(null)

const descargarPDF = async () => {
  if (!invoiceRef.value) return

  try {
    const canvas = await html2canvas(invoiceRef.value, {
      scale: 1.5,
      backgroundColor: '#ffffff',
      logging: false
    })

    const imgData = canvas.toDataURL('image/png')
    const pdf = new jsPDF('p', 'mm', [80, 200])
    const pdfWidth = pdf.internal.pageSize.getWidth()
    const pageHeight = pdf.internal.pageSize.getHeight()
    const imgHeight = (canvas.height * pdfWidth) / canvas.width

    if (imgHeight > pageHeight) {
      const scale = pageHeight / imgHeight
      pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth * scale, imgHeight * scale)
    } else {
      pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, imgHeight)
    }

    pdf.save(`Factura_${invoiceNumber.value}.pdf`)
    cart.value = []
    notificar('Factura descargada. Carrito limpiado.', 'success')
  } catch (error) {
    notificar('Error al generar el PDF', 'error')
  }
}

const nuevoPedido = () => {
  cart.value = []
  currentView.value = 'menu'
}
</script>

<style scoped>
.container {
  min-height: 100vh;
}
</style>
