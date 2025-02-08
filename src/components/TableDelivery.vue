<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'
import { mdiEye, mdiTrashCan, mdiPlus } from '@mdi/js'
import CardBoxModal from '@/components/CardBoxModal.vue'
import BaseLevel from '@/components/BaseLevel.vue'
import BaseButtons from '@/components/BaseButtons.vue'
import BaseButton from '@/components/BaseButton.vue'

defineProps({
  checkable: Boolean
})

const url = "https://saber-traders-backend-d1gm.vercel.app";

const isModalActive = ref(false)
const isModalDangerActive = ref(false)
const isAddModalActive = ref(false)
const isAddTruckModalActive = ref(false)
const selectedTransport = ref(null)
const newTransport = ref({
  truckNo: '',
  date: '',
  description: '',
  taka: '',
  driverSalary: '',
  remainingTaka: ''
})

const newTruck = ref({
  truckNo: '',
})
const perPage = ref(5)
const currentPage = ref(0)
const transportData = ref([])
const truckData = ref([])

const fetchTransportData = async () => {
  try {
    const response = await axios.get(`${url}/traders`)
    transportData.value = response.data.data
  } catch (error) {
    console.error('Error fetching data:', error)
  }
}

const fetchTruckData = async()=>{
  try {
    const response = await axios.get(`${url}/trucks`)
    truckData.value = response.data.data
  } catch (error) {
    console.error('Error fetching data:', error)
  }
}

const validateTransport = (transport) => {
  return transport.truckNo && transport.date && transport.description && transport.taka && transport.driverSalary && transport.remainingTaka
}

const viewTransport = (transport) => {
  selectedTransport.value = transport
  isModalActive.value = true
}

const editTransport = async () => {
  if (!validateTransport(selectedTransport.value)) {
    alert('Please fill all required fields: truckNo, date, description, taka, driverSalary, remainingTaka')
    return
  }
  try {
    await axios.put(`${url}/traders/${selectedTransport.value._id}`, selectedTransport.value)
    fetchTransportData()
    isModalActive.value = false
  } catch (error) {
    console.error('Error editing data:', error)
  }
}

const addTruck = async () => {

  try {
    await axios.post(`${url}/trucks`, newTruck.value)
    fetchTruckData()
    isAddTruckModalActive.value = false
    newTruck.value = { truckNo: '', }
  } catch (error) {
    console.error('Error adding data:', error)
  }
}

const addTransport = async () => {
  if (!validateTransport(newTransport.value)) {
    alert('Please fill all required fields: truckNo, date, description, taka, driverSalary, remainingTaka')
    return
  }
  try {
    await axios.post(`${url}/traders`, newTransport.value)
    fetchTransportData()
    isAddModalActive.value = false
    newTransport.value = { truckNo: '', date: '', description: '', taka: '', driverSalary: '', remainingTaka: '' }
  } catch (error) {
    console.error('Error adding data:', error)
  }
}

const deleteTransport = async (id) => {
  try {
    await axios.delete(`${url}/traders/${id}`)
    fetchTransportData()
  } catch (error) {
    console.error('Error deleting data:', error)
  }
}

onMounted(fetchTransportData)
onMounted(fetchTruckData)

console.log(truckData)
const itemsPaginated = computed(() =>
  transportData.value.slice(perPage.value * currentPage.value, perPage.value * (currentPage.value + 1))
)

const numPages = computed(() => Math.ceil(transportData.value.length / perPage.value))
const currentPageHuman = computed(() => currentPage.value + 1)
const pagesList = computed(() => Array.from({ length: numPages.value }, (_, i) => i))
</script>

<template>

  <BaseButton color="success" :icon="mdiPlus" label="Add Trader" @click="isAddModalActive = true" />

  <BaseButton class="mx-2" color="info" :icon="mdiPlus" label="Add Truck" @click="isAddTruckModalActive = true"/>

  <CardBoxModal v-model="isAddModalActive" title="Add New Transport">
    <select v-model="newTransport.truckNo" class="input" required>
    <option value="" disabled>Select Truck No</option>
    <option v-for="truck in truckData" :key="truck._id" :value="truck.truckNo">
      {{ truck.truckNo }}
    </option>
  </select>
      <input v-model="newTransport.date" type="date" class="input" placeholder="Date" required />
      <input v-model="newTransport.description" type="text" class="input" placeholder="Description" required />

    <div class="grid grid-cols-2 gap-2">
      <input v-model="newTransport.quantityOfCementBagRod" type="number" class="input" placeholder="Quantity of Cement Bag/Rod" />
      <input v-model="newTransport.priceRate" type="number" class="input" placeholder="Price Rate" />
      <input v-model="newTransport.taka" type="number" class="input" placeholder="Taka" required />
      <input v-model="newTransport.driverSalary" type="number" class="input" placeholder="Driver Salary" required />
      <input v-model="newTransport.fuelExpense" type="number" class="input" placeholder="Fuel Expense" />
      <input v-model="newTransport.labourGratuity" type="number" class="input" placeholder="Labour Gratuity" />
      <input v-model="newTransport.toll" type="number" class="input" placeholder="Toll" />
      <input v-model="newTransport.transportCost" type="number" class="input" placeholder="Transport Cost" />
      <input v-model="newTransport.remainingTaka" type="number" class="input" placeholder="Remaining Taka" required />
    </div>


    <BaseButtons>
      <BaseButton color="primary" label="Add" @click="addTransport" />
      <BaseButton color="secondary" label="Cancel" @click="isAddModalActive = false" />
    </BaseButtons>
  </CardBoxModal>

  <CardBoxModal v-model="isAddTruckModalActive" title="Add New Truck">

      <input v-model="newTruck.truckNo" type="text" class="input" placeholder="Truck No." required />
    <BaseButtons>
      <BaseButton color="primary" label="Add" @click="addTruck" />
      <BaseButton color="secondary" label="Cancel" @click="isAddTruckModalActive = false" />
    </BaseButtons>
  </CardBoxModal>


  <CardBoxModal v-model="isModalActive" title="Edit Transport Details">
    <div  v-if="selectedTransport">

      <select v-model="newTransport.truckNo" class="input" required>
    <option value="" disabled>Select Truck No</option>
    <option v-for="truck in truckData" :key="truck._id" :value="truck.truckNo">
      {{ truck.truckNo }}
    </option>
    </select>

      <input v-model="selectedTransport.date" type="date" class="input" placeholder="Date" required />
      <input v-model="selectedTransport.description" type="text" class="input" placeholder="Description" required />

      <div class="grid grid-cols-2 gap-2">
        <input v-model="selectedTransport.quantityOfCementBagRod" type="number" class="col-6 input" placeholder="Quantity of Cement Bag/Rod" />
      <input  v-model="selectedTransport.priceRate" type="number" class="col-6 input" placeholder="Price Rate" />
      <input v-model="selectedTransport.taka" type="number" class="input" placeholder="Taka" required />
      <input v-model="selectedTransport.driverSalary" type="number" class="col-6 input" placeholder="Driver Salary" required />
      <input v-model="selectedTransport.fuelExpense" type="number" class="col-6 input" placeholder="Fuel Expense" />
      <input v-model="selectedTransport.labourGratuity" type="number" class="col-6 input" placeholder="Labour Gratuity" />
      <input v-model="selectedTransport.toll" type="number" class="col-6 input" placeholder="Toll" />
      <input v-model="selectedTransport.transportCost" type="number" class="col-6 input" placeholder="Transport Cost" />
      <input v-model="selectedTransport.remainingTaka" type="number" class="col-6 input" placeholder="Remaining Taka" required />
      </div>

      <BaseButtons>
        <BaseButton color="primary" label="Save Changes" @click="editTransport" />
        <BaseButton color="secondary" label="Cancel" @click="isModalActive = false" />
      </BaseButtons>
    </div>
  </CardBoxModal>

  <CardBoxModal v-model="isModalDangerActive" title="Please confirm" button="danger" has-cancel>
    <p>Are you sure you want to delete this record?</p>
    <BaseButtons>
      <BaseButton color="danger" label="Confirm" @click="deleteTransport(selectedTransport._id); isModalDangerActive = false" />
      <BaseButton color="secondary" label="Cancel" @click="isModalDangerActive = false" />
    </BaseButtons>
  </CardBoxModal>

  <table>
    <thead>
      <tr>
        <th v-if="checkable" />
        <th>Truck No</th>
        <th>Date</th>
        <th>Description</th>
        <th>Taka</th>
        <th>Driver Salary</th>
        <th>Remaining Taka</th>
        <th>Actions</th>
        <th />
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in itemsPaginated" :key="item._id">
        <TableCheckboxCell v-if="checkable" @checked="checked($event, item)" />
        <td data-label="Name">
          {{ item.truckNo }}
        </td>
        <td data-label="Date">
          {{ item.date }}
        </td>
        <td data-label="description">
          {{ item.description }}
        </td>
        <td data-label="taka">
          {{ item.taka }}
        </td>
        <td data-label="DriverSalary">
          {{ item.driverSalary }}
        </td>
        <td data-label="RemainingTaka">
          {{ item.remainingTaka }}
        </td>
        <td class="before:hidden lg:w-1 whitespace-nowrap">
          <BaseButtons type="justify-start lg:justify-end" no-wrap>
            <BaseButton color="info" :icon="mdiEye" small @click="viewTransport(item)" />
            <BaseButton color="danger" :icon="mdiTrashCan" small @click="() => { selectedTransport = item; isModalDangerActive = true; }" />
          </BaseButtons>
        </td>
      </tr>
    </tbody>
  </table>

  <div class="p-3 border-t">
    <BaseLevel>
      <BaseButtons>
        <BaseButton
          v-for="page in pagesList"
          :key="page"
          :active="page === currentPage"
          :label="page + 1"
          :color="page === currentPage ? 'lightDark' : 'whiteDark'"
          small
          @click="currentPage = page"
        />
      </BaseButtons>
      <small>Page {{ currentPageHuman }} of {{ numPages }}</small>
    </BaseLevel>
  </div>
</template>

<style>
.input {
  display: block;
  width: 100%;
  padding: 10px;
  margin: 5px 0 15px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.form-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
</style>
