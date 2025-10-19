<template>
  <div class="tabla-productos">
    <Card>
      <template #title>Listado de Productos</template>
      <template #content>
        <div class="contenedor-tabla">
          <DataTable
            :value="productosPaginados"
            :loading="cargando"
            paginator
            :rows="filasPorPagina"
            :rowsPerPageOptions="[5, 10, 20, 50]"
            :totalRecords="productos.length"
            tableStyle="min-width: 50rem"
            class="p-datatable-sm"
          >
            <Column field="id" header="ID" style="width: 10%"></Column>
            <Column field="nombre" header="Nombre" style="width: 25%"></Column>
            <Column field="descripcion" header="Descripción" style="width: 30%">
              <template #body="datosFila">
                {{ datosFila.data.descripcion || 'Sin descripción' }}
              </template>
            </Column>
            <Column field="precio" header="Precio" style="width: 15%">
              <template #body="datosFila">
                {{ formatearPrecio(datosFila.data.precio) }}
              </template>
            </Column>
            <Column header="Acciones" style="width: 20%">
              <template #body="datosFila">
                <Button
                  icon="pi pi-pencil"
                  class="p-button-text"
                  @click="emitirEditar(datosFila.data)"
                />
                <Button
                  icon="pi pi-trash"
                  class="p-button-text"
                  @click="emitirEliminar(datosFila.data.id)"
                />

                <Button
                  label="Generar Reporte PDF"
                  icon="pi pi-file-pdf"
                  @click="generarPDF"
                  severity="danger"
                />
              </template>
            </Column>

            <template #empty>
              <div class="estado-vacio">
                <i class="pi pi-inbox" style="font-size: 2rem"></i>
                <p>No se encontraron productos en la DB</p>
              </div>
            </template>

            <template #loading>
              <div class="estado-cargando">
                <i class="pi pi-spinner pi-spin" style="font-size: 2rem"></i>
                <p>Cargando productos...</p>
              </div>
            </template>
          </DataTable>
        </div>

        <div class="informacion-tabla">
          <span class="p-text-secondary">
            Mostrando {{ mostrarDesde }} a {{ mostrarHasta }} de {{ productos.length }} productos
          </span>
        </div>
      </template>
    </Card>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useProductoService } from '@/service/api'
import jsPDF from 'jspdf'
import html2canvas from 'html2canvas'
const { obtenerProductos } = useProductoService()

// 1. Definir los eventos que este componente emitirá al padre
const emit = defineEmits(['editar-producto', 'eliminar-producto'])

// 2. Función para emitir el evento de Edición
const emitirEditar = (producto) => {
  console.log('Editando producto:', producto.nombre)
  emit('editar-producto', producto) // Emite el producto completo
}

// 3. Función para emitir el evento de Eliminación
const emitirEliminar = (id) => {
  console.log('Confirmar eliminación de ID:', id)
  emit('eliminar-producto', id) // Emite solo el ID

  const generarPDF = () => {
    // Lógica para generar el PDF del listado de productos
    console.log('Generando reporte PDF de productos...')

    const generarPDF = () => {
      // 1. Obtener el elemento HTML que quieres exportar
      const element = document.getElementById('contenido-a-exportar')

      if (!element) {
        console.error("No se encontró el elemento con id 'contenido-a-exportar'")
        return
      }

      // Dimensiones de la imagen
      const imgWidth = 210 // Ancho A4 en mm
      const pageHeight = 295 // Alto A4 en mm
      const imgHeight = (canvas.height * imgWidth) / canvas.width
      let heightLeft = imgHeight

      let position = 0

      // 3. Agregar la imagen al PDF
      pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight)
      heightLeft -= pageHeight

      // 4. Manejar el contenido que desborda la página (múltiples páginas)
      while (heightLeft >= 0) {
        position = heightLeft - imgHeight
        pdf.addPage()
        pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight)
        heightLeft -= pageHeight
      }

      // 5. Guardar el PDF
      pdf.save('Reporte_HTML_Exportado.pdf')
    }
  }
  // Aquí puedes integrar una librería como jsPDF para crear el PDF
}

// Variables reactivas
const productos = ref([])
const cargando = ref(false)
const filasPorPagina = ref(5)

// Computed properties
const productosPaginados = computed(() => {
  return productos.value
})

// Paginador desde
const mostrarDesde = computed(() => {
  return 1
})

// Paginador hasta
const mostrarHasta = computed(() => {
  return productos.value.length
})

// Método para convertir el precio
const formatearPrecio = (precio) => {
  return new Intl.NumberFormat('es-MX', {
    style: 'currency',
    currency: 'MXN',
  }).format(precio || 0)
}

// Cargar productos
const cargarProductos = async () => {
  cargando.value = true
  try {
    const respuesta = await obtenerProductos()

    // Manejar diferentes formatos de respuesta
    if (Array.isArray(respuesta)) {
      productos.value = respuesta
    } else if (respuesta && Array.isArray(respuesta.data)) {
      productos.value = respuesta.data
    } else if (respuesta && respuesta.productos && Array.isArray(respuesta.productos)) {
      productos.value = respuesta.productos
    } else if (respuesta && respuesta.items && Array.isArray(respuesta.items)) {
      productos.value = respuesta.items
    } else {
      console.error('Estructura de respuesta no reconocida:', respuesta)
      productos.value = []
    }
  } catch (error) {
    console.error('Error al cargar productos:', error)
    productos.value = []
  } finally {
    cargando.value = false
  }
}

// Exponer método al componente padre
defineExpose({
  cargarProductos,
})

// Ciclo de vida de carga del componente
onMounted(() => {
  cargarProductos()
})
</script>
