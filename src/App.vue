<template>
  <Toast />
  <div class="bg-slate-300 h-screen w-screen overflow-hidden align-text-top text-center">
      <h1 class="mb-5 my-4 mx-5 text-2xl font-bold text-blue-600">Crud con Spring Boot + Vue3</h1>
   
    <div class="flex justify-center items-start h-full">
      <div class="card">
        <Toolbar class="mb-4">
          <template #start>
            <Button label="Nuevo" icon="pi pi-plus"  class="mr-2" @click="openNew" />
          </template>
        </Toolbar>

        <DataTable ref="dt" :value="estudiantes" v-model:selection="selectedEstudiante" dataKey="id" :paginator="true"
          :rows="5" currentPageReportTemplate="Mostrando {first} a {last} de {totalRecords} estudiantes">
          <template #header>
            <div class="flex flex-wrap gap-2 align-items-center justify-content-between">
              <h4 class="m-0">Administrar Estudiantes</h4>
              <div>
                <IconField iconPosition="left">
                  <InputIcon>
                    <i class="pi pi-search" />
                  </InputIcon>
                  <InputText v-model="globalFilter" @input="handleSearch" placeholder="Buscar..." />
                </IconField>
              </div>
            </div>
          </template>

          <Column selectionMode="multiple" style="width: 3rem" :exportable="false"></Column>
          <Column field="id" header="ID" sortable style="min-width: 6rem"></Column>
          <Column field="nombre" header="Nombre" sortable style="min-width: 12rem"></Column>
          <Column field="direccion" header="Dirección" sortable style="min-width: 16rem"></Column>
          <Column field="telefono" header="Teléfono" sortable style="min-width: 10rem"></Column>
          <Column :exportable="false" style="min-width: 8rem">
            <template #body="slotProps">
              <Button icon="pi pi-pencil" class="mr-2" @click="editEstudiante(slotProps.data)" />
              <Button icon="pi pi-trash" class="mr-2" @click="confirmDeleteEstudiante(slotProps.data)" />
            </template>
          </Column>
        </DataTable>
      </div>

      <!-- Diálogos -->
      <Dialog v-model:visible="estudianteDialog" :style="{ width: '450px' }" header="Detalles del Estudiante"
        :modal="true">
        <div class="field">
          <label for="nombre">Nombre</label>
          <InputText id="nombre" v-model.trim="estudiante.nombre" required :invalid="submitted && !estudiante.nombre" />
          <small class="p-error" v-if="submitted && !estudiante.nombre">El nombre es requerido.</small>
        </div>
        <div class="field">
          <label for="direccion">Dirección</label>
          <InputText id="direccion" v-model.trim="estudiante.direccion" />
        </div>
        <div class="field">
          <label for="telefono">Teléfono</label>
          <InputText id="telefono" v-model.trim="estudiante.telefono" />
        </div>
        <template #footer>
          <Button label="Cancelar" icon="pi pi-times" text @click="hideDialog" />
          <Button label="Guardar" icon="pi pi-check" text @click="saveEstudiante" />
        </template>
      </Dialog>

      <Dialog v-model:visible="deleteEstudianteDialog" :style="{ width: '450px' }" header="Confirmar" :modal="true">
        <div class="confirmation-content">
          <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem"></i>
          <span v-if="estudiante">¿Estás seguro de que quieres eliminar <b>{{ estudiante.nombre }}</b>?</span>
        </div>
        <template #footer>
          <Button label="No" icon="pi pi-times" text @click="deleteEstudianteDialog = false" />
          <Button label="Sí" icon="pi pi-check" text @click="deleteEstudiante" />
        </template>
      </Dialog>

      <Dialog v-model:visible="deleteEstudiantesDialog" :style="{ width: '450px' }" header="Confirmar" :modal="true">
        <div class="confirmation-content">
          <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem"></i>
          <span v-if="estudiantes && selectedEstudiante">¿Estás seguro de que quieres eliminar los estudiantes
            seleccionados?</span>
        </div>
        <template #footer>
          <Button label="No" icon="pi pi-times" text @click="deleteEstudiantesDialog = false" />
          <Button label="Sí" icon="pi pi-check" text @click="deleteSelectedEstudiantes" />
        </template>
      </Dialog>
    </div>
  </div>

</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useToast } from "primevue/usetoast";
const toast = useToast();
const showSuccess = () => {
    toast.add({ severity: 'success', summary: 'Success Message', detail: 'Message Content', life: 3000 });
};
const estudiantes = ref([]); // Tu lista de estudiantes
const selectedEstudiante = ref(null); // Estudiante seleccionado
const estudianteDialog = ref(false); // Mostrar/ocultar diálogo de detalles del estudiante
const deleteEstudianteDialog = ref(false); // Mostrar/ocultar diálogo de confirmación de eliminación de estudiante
const deleteEstudiantesDialog = ref(false); // Mostrar/ocultar diálogo de confirmación de eliminación de estudiantes seleccionados
const estudiante = ref({}); // Objeto de estudiante para edición/agregación
const submitted = ref(false); // Indicador de formulario enviado
const globalFilter = ref(''); // Filtro global de búsqueda

function openNew() {
  estudiante.value = {}; // Limpiar el objeto de estudiante
  submitted.value = false; // Reiniciar el indicador de formulario enviado
  estudianteDialog.value = true; // Mostrar diálogo de detalles del estudiante
}

function hideDialog() {
  estudianteDialog.value = false; // Ocultar diálogo de detalles del estudiante
  submitted.value = false; // Reiniciar el indicador de formulario enviado
}

function editEstudiante(selectedEstudiante) {
  // Asignar los datos del estudiante seleccionado al objeto estudiante
  estudiante.value = { ...selectedEstudiante };
  // Mostrar el diálogo de detalles del estudiante
  estudianteDialog.value = true;
}

async function saveEstudiante() {
  submitted.value = true; // Marcar formulario como enviado

  try {
    if (!estudiante.value.id) {
      // Si el ID del estudiante no está presente, significa que estamos creando un nuevo estudiante
      const response = await fetch('http://localhost:8080/api/v1/estudiante', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(estudiante.value),
      });
      showSuccess();
      if (!response.ok) {
        throw new Error('Failed to add new estudiante');
      }
    } else {
      // Si el ID del estudiante está presente, significa que estamos editando un estudiante existente
      const response = await fetch(`http://localhost:8080/api/v1/estudiante/${estudiante.value.id}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(estudiante.value),
      });
      
      if (!response.ok) {
        throw new Error('Failed to update estudiante');
      }
    }

    // Una vez que la solicitud se haya completado con éxito, cierra el diálogo y vuelve a cargar la lista de estudiantes
    hideDialog();
    fetchEstudiantes();
  } catch (error) {
    console.error(error);
    // Aquí puedes mostrar un mensaje de error al usuario si lo deseas
  }
}

function confirmDeleteEstudiante(selectedEstudiante) {
  estudiante.value = selectedEstudiante; // Asignar el estudiante seleccionado para eliminar
  deleteEstudianteDialog.value = true; // Mostrar diálogo de confirmación de eliminación de estudiante
}

async function deleteEstudiante() {
  try {
    // Verificar que el objeto estudiante tenga una propiedad id definida
    if (!estudiante.value || !estudiante.value.id) {
      throw new Error('No se ha seleccionado ningún estudiante para eliminar');
    }

    // Obtener el ID del estudiante a eliminar
    const estudianteId = estudiante.value.id;

    // Enviar una solicitud DELETE al endpoint correspondiente con el ID del estudiante
    const response = await fetch(`http://localhost:8080/api/v1/estudiante/${estudianteId}`, {
      method: 'DELETE',
    });

    if (!response.ok) {
      throw new Error('Failed to delete estudiante');
    }

    // Una vez que el estudiante se ha eliminado exitosamente, cerrar el diálogo y recargar la lista de estudiantes
    deleteEstudianteDialog.value = false;
    fetchEstudiantes();
  } catch (error) {
    console.error(error);
    // Manejar cualquier error que pueda ocurrir durante la eliminación del estudiante
  }
}


function deleteSelectedEstudiantes() {

  deleteEstudiantesDialog.value = false;
}
// Función para obtener la lista de estudiantes desde la API
async function fetchEstudiantes() {
  try {
    const response = await fetch('http://localhost:8080/api/v1/estudiante');
    if (!response.ok) {
      throw new Error('Failed to fetch estudiantes');
    }
    const data = await response.json();
    estudiantes.value = data; // Actualizar la lista de estudiantes con los datos obtenidos de la API
  } catch (error) {
    console.error(error);
  }
}
// Función para manejar la búsqueda
// Función para manejar la búsqueda
async function handleSearch() {
  try {
    const response = await fetch(`http://localhost:8080/api/v1/estudiante/nombre/${globalFilter.value}`);
    if (!response.ok) {
      throw new Error('Failed to fetch estudiantes');
    }
    const data = await response.json();

    // Verifica si data es un objeto o un array
    if (Array.isArray(data)) {
      estudiantes.value = data; // Actualizar la lista de estudiantes con los datos obtenidos de la API
    } else if (typeof data === 'object') {
      estudiantes.value = [data]; // Convertir el objeto en un array con un solo elemento
    } else {
      throw new Error('Invalid data format');
    }
  } catch (error) {
    console.error(error);
  }
}
// Llamar a fetchEstudiantes cuando el componente se monta
onMounted(() => {
  fetchEstudiantes();
});
</script>