v-if="currentviw ===menu"solo muestra todo este bloque si el usuario esta en la pestaña menu

v for=products..:es un bluque,por cada objeto que en en mi lista productos,vui copia y pega todo el bloque de codigo html

:key es el dni para que vio no se pierda al actualizar

:src le dice a la imagen que foto mostrar y las llaves escriben el nombre y el precio cada tarjeta es unica porque usa los datos de su propio objeeto product

:disable desactiva el boton de compra si avaliable e o o menos.es la logica que impide que el usuario cometa el error al pedir algo que no hay

en una palabra:mapeo de dtus datos(js) a elementos viduales(html)

---------------------------------------------
agregar producto.

v-else-if="currentView === 'addProduct'"

este formulario solo aparece si la variable currentviw dice exactamente addproducst.si estoy viendo otro menu este bloque no va aparecer

v-model:conecta el input con tu variable en js 



-------------------------------

carrito:
navegacion por estado:
v-else-if="currentView === 'cart'":si el usuario cambio la vista a cart apaga el menu y activa esta seccion


control de contenido vacio
v-if="cart.length === 0"
antes de dibujar la lista de compras pregunte hay algo en la bolsa?
si length es cero muestra el carrito vacio

si es mayor a cero,este bloque se ignora y pasa al siguiente donde estarian lo productos

------------------------------------------------

v-else
si la longitud no es cero ignora el avido y pinta de una la lista

v-for="item in cart"realiza un mapeo de los objetos contenidos en el array cart hacia elementos del DOM

---------------------------

v-else-if="currentView === 'invoice'"

esta vista solo se activa cuando el ususario hace clic en finalizar compra oculta el carrito ybel menu para mostrar unicamente la factura


v-for="item in cart":
reutilizacion del estado del carrito para una vista solo de lectura


en el carrito:usa v-for para editar(sumar/quitar)
en la factura:uso vfor para mostra el resumen final



------------------------------------------------

const newid=math.max:busca el id mas alto que exite y le suma uno para el nuevo producto

push:
es la accion final de insertar el nuevo producto en tu lista oficial para que aparezca en la pantalla

------------------------------------------------
findindex:
es para saber en que posicion esta exactamente el producto en la lista


.find:nos da el objeto completo que estoy buscando





splice:
qita o mete cosas en cualquier parte de la lista

