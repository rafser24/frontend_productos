<template>
  <div class="nuevo-producto">
    <Button label="Registrar" icon="pi pi-plus" @click="abrirModalRegistro" />

    <Dialog
      v-model:visible="visible"
      modal
      :header="modoEdicion ? 'Editar producto' : 'Registrar nuevo producto'"
      :style="{ with: '450px' }"
      :draggable="false"
    >
      <div class="formulario-producto">
        <div class="campo">
          <label for="nombre" class="etiqueta">Nombre del producto*:</label>
          <InputText
            v-model="producto.nombre"
            id="nombre"
            class="input-completo"
            placeholder="Ej: Lapto HP"
          />
        </div>

        <div class="campo">
          <label for="descripcion" class="etiqueta">Descripción del producto*:</label>
          <Textarea
            v-model="producto.descripcion"
            id="descripcion"
            class="input-completo"
            placeholder="Ej: Explica el detalle del producto"
            rows="3"
            autoResize
          />
        </div>

        <div class="campo">
          <label for="precio" class="etiqueta">Precio*:</label>
          <InputNumber v-model="producto.precio" id="precio" class="input-completo" />
        </div>
      </div>

      <template #footer>
        <div class="pie-modal">
          <Button
            label="Cancelar"
            icon="pi pi-times"
            class="boton-secundario"
            style="margin-right: 0.5rem"
            severity="secondary"
            @click="cerrarModal"
          />

          <Button
            :label="modoEdicion ? 'Actualizar' : 'Guardar'"
            :icon="modoEdicion ? 'pi pi-pencil' : 'pi pi-check'"
            class="boton-primario"
            @click="modoEdicion ? editarProducto() : guardarProducto()"
          />
        </div>
      </template>
    </Dialog>

    <Dialog
      v-model:visible="visibleEliminar"
      modal
      header="Confirmar Eliminación"
      :style="{ width: '350px' }"
      :draggable="false"
    >
      <div class="p-d-flex p-ai-center">
        <i class="pi pi-exclamation-triangle p-mr-3" style="font-size: 2rem; color: orange"></i>
        <span>¿Estás seguro de que deseas eliminar este producto?</span>
      </div>
      <template #footer>
        <Button
          label="No"
          icon="pi pi-times"
          class="p-button-text"
          @click="visibleEliminar = false"
        />
        <Button
          label="Sí"
          icon="pi pi-check"
          severity="danger"
          @click="eliminarProducto"
          autofocus
        />
      </template>
    </Dialog>

    <Toast />
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
// Se asume que estos servicios existen y ahora incluyen updateProducto y deleteProducto
import { useProductoService } from '@/service/api'

import { useToast } from 'primevue/usetoast'
import Toast from 'primevue/toast'
const toast = useToast()

// El componente padre debe llamar a abrirModalEdicion(producto) y confirmarEliminar(id)
const emit = defineEmits(['producto-creado', 'producto-editado', 'producto-eliminado'])

const visible = ref(false) // Controla el modal de Registro/Edición
const visibleEliminar = ref(false) // Controla el modal de Eliminación
const modoEdicion = ref(false) // Estado para saber si el modal está en modo edición
const productoIdAEliminar = ref(null) // Almacena el ID del producto a eliminar

const producto = reactive({
  id: null, // Agregamos id para edición
  nombre: '',
  descripcion: '',
  precio: null,
})

const errores = reactive({
  nombre: '',
  descripcion: '',
  precio: '',
})

// Desestructurar todos los servicios necesarios
const { createProducto, updateProducto, deleteProducto } = useProductoService()

// --- Funciones para el Modal de Registro/Edición ---

const abrirModalRegistro = () => {
  modoEdicion.value = false // Modo Registro
  resetearFormulario()
  visible.value = true
}

// Función que el componente padre usará para iniciar la edición
const abrirModalEdicion = (prod) => {
  modoEdicion.value = true // Modo Edición
  // Copiar las propiedades del producto seleccionado al estado reactivo
  producto.id = prod.id
  producto.nombre = prod.nombre
  producto.descripcion = prod.descripcion
  producto.precio = prod.precio
  visible.value = true
}

const guardarProducto = async () => {
  try {
    // ... Lógica de validación (no incluida pero recomendada) ...

    const datosProducto = {
      nombre: producto.nombre.trim(),
      descripcion: producto.descripcion.trim(),
      precio: parseFloat(producto.precio),
    }

    const respuesta = await createProducto(datosProducto) // Llamar a API para crear

    toast.add({
      severity: 'success',
      summary: 'Éxito',
      detail: `Producto ${producto.nombre} almacenado`,
      life: 5000,
    })

    emit('producto-creado') // Evento para recargar la tabla
    cerrarModal()
  } catch (error) {
    console.error('Error al guardar producto:', error)
    toast.add({
      severity: 'error',
      summary: 'Error',
      detail: 'No se pudo crear el registro',
      life: 5000,
    })
  }
}

const editarProducto = async () => {
  if (!producto.id) return // Asegurar que haya un ID

  try {
    // ... Lógica de validación (no incluida pero recomendada) ...

    const datosProducto = {
      id: producto.id,
      nombre: producto.nombre.trim(),
      descripcion: producto.descripcion.trim(),
      precio: parseFloat(producto.precio),
    }

    const respuesta = await updateProducto(datosProducto.id, datosProducto) // Llamar a API para actualizar

    toast.add({
      severity: 'success',
      summary: 'Éxito',
      detail: `Producto ${producto.nombre} actualizado`,
      life: 5000,
    })

    emit('producto-editado') // Evento para recargar la tabla
    cerrarModal()
  } catch (error) {
    console.error('Error al editar producto:', error)
    toast.add({
      severity: 'error',
      summary: 'Error',
      detail: 'No se pudo actualizar el registro',
      life: 5000,
    })
  }
}

// --- Funciones para la Eliminación ---

// Función que el componente padre usará para iniciar la eliminación
const confirmarEliminar = (productoId) => {
  productoIdAEliminar.value = productoId
  visibleEliminar.value = true
}

const eliminarProducto = async () => {
  if (!productoIdAEliminar.value) return

  try {
    await deleteProducto(productoIdAEliminar.value) // Llamar a API para eliminar

    toast.add({
      severity: 'success',
      summary: 'Éxito',
      detail: 'Producto eliminado correctamente',
      life: 5000,
    })

    emit('producto-eliminado') // Evento para recargar la tabla
    visibleEliminar.value = false
    productoIdAEliminar.value = null
  } catch (error) {
    console.error('Error al eliminar producto:', error)
    toast.add({
      severity: 'error',
      summary: 'Error',
      detail: 'No se pudo eliminar el registro',
      life: 5000,
    })
  }
}

// --- Funciones Auxiliares ---

const resetearFormulario = () => {
  producto.id = null
  producto.nombre = ''
  producto.descripcion = ''
  producto.precio = null
  Object.keys(errores).forEach((key) => (errores[key] = ''))
}

const cerrarModal = () => {
  visible.value = false
  // Opcional: resetear el formulario al cerrar si no se guarda
  resetearFormulario()
}

// Exportar las funciones de abrir/confirmar para que el componente padre pueda llamarlas
defineExpose({
  abrirModalEdicion,
  confirmarEliminar,
})
</script>
