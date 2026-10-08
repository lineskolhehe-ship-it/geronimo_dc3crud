<script setup>
import { ref, computed, onMounted } from 'vue'

const foods = ref([])

const foodName = ref('')
const category = ref('')
const description = ref('')
const price = ref('')
const available = ref(true)

const editingId = ref(null)
const search = ref('')
const filterCategory = ref('All')

const categories = [
  'Burger',
  'Pizza',
  'Chicken',
  'Pasta',
  'Drinks',
  'Desserts'
]

onMounted(() => {
  const savedFoods = localStorage.getItem('foods')

  if (savedFoods) {
    foods.value = JSON.parse(savedFoods)
  }
})


const saveFoods = () => {
  localStorage.setItem('foods', JSON.stringify(foods.value))
}


const saveFood = () => {
  if (
    !foodName.value ||
    !category.value ||
    !description.value ||
    !price.value
  ) {
    alert('Please complete all fields.')
    return
  }

  if (editingId.value === null) {
    const newFood = {
      id: Date.now(),
      name: foodName.value,
      category: category.value,
      description: description.value,
      price: Number(price.value),
      available: available.value
    }

    foods.value.push(newFood)
  } else {
    const food = foods.value.find(
      item => item.id === editingId.value
    )

    if (food) {
      food.name = foodName.value
      food.category = category.value
      food.description = description.value
      food.price = Number(price.value)
      food.available = available.value
    }

    editingId.value = null
  }

  saveFoods()
  clearForm()
}
const editFood = (food) => {
  editingId.value = food.id

  foodName.value = food.name
  category.value = food.category
  description.value = food.description
  price.value = food.price
  available.value = food.available
}

const deleteFood = (id) => {
  if (confirm('Are you sure you want to delete this food?')) {
    foods.value = foods.value.filter(
      food => food.id !== id
    )

    saveFoods()
  }
}

const clearForm = () => {
  foodName.value = ''
  category.value = ''
  description.value = ''
  price.value = ''
  available.value = true
  editingId.value = null
}


const filteredFoods = computed(() => {
  return foods.value.filter(food => {
    const matchesSearch =
      food.name.toLowerCase().includes(search.value.toLowerCase())

    const matchesCategory =
      filterCategory.value === 'All' ||
      food.category === filterCategory.value

    return matchesSearch && matchesCategory
  })
})
</script>

<template>
  <div class="container">

    <header>
      <h1>Food Ordering System</h1>
      <p>Food Management CRUD Application</p>
    </header>

    <section class="card">

      <h2>
        {{ editingId === null ? 'Add Food' : 'Edit Food' }}
      </h2>

      <form @submit.prevent="saveFood">

        <div class="form-group">
          <label>Food Name</label>

          <input
            v-model="foodName"
            type="text"
            placeholder="Enter food name"
          />
        </div>

        <div class="form-group">
          <label>Category</label>

          <select v-model="category">
            <option value="">Select category</option>

            <option
              v-for="item in categories"
              :key="item"
              :value="item"
            >
              {{ item }}
            </option>
          </select>
        </div>

        <div class="form-group">
          <label>Description</label>

          <textarea
            v-model="description"
            placeholder="Enter food description"
          ></textarea>
        </div>

        <div class="form-group">
          <label>Price</label>

          <input
            v-model="price"
            type="number"
            placeholder="Enter price"
          />
        </div>

        <div class="checkbox">
          <input
            v-model="available"
            type="checkbox"
            id="available"
          />

          <label for="available">
            Available
          </label>
        </div>

        <button class="btn-primary" type="submit">
          {{ editingId === null ? 'Add Food' : 'Update Food' }}
        </button>

        <button
          v-if="editingId !== null"
          class="btn-secondary"
          type="button"
          @click="clearForm"
        >
          Cancel
        </button>

      </form>
    </section>

  
    <section class="card">

      <div class="list-header">

        <h2>Food List</h2>

        <div class="filters">

          <input
            v-model="search"
            type="text"
            placeholder="Search food..."
          />

          <select v-model="filterCategory">
            <option value="All">All Categories</option>

            <option
              v-for="item in categories"
              :key="item"
              :value="item"
            >
              {{ item }}
            </option>
          </select>

        </div>

      </div>

      <div v-if="filteredFoods.length === 0" class="empty">
        No food items found.
      </div>

      <div v-else class="table-container">

        <table>

          <thead>
            <tr>
              <th>ID</th>
              <th>Food Name</th>
              <th>Category</th>
              <th>Description</th>
              <th>Price</th>
              <th>Status</th>
              <th>Actions</th>
            </tr>
          </thead>

          <tbody>

            <tr
              v-for="food in filteredFoods"
              :key="food.id"
            >

              <td>{{ food.id }}</td>

              <td>
                <strong>{{ food.name }}</strong>
              </td>

              <td>
                {{ food.category }}
              </td>

              <td>
                {{ food.description }}
              </td>

              <td>
                ₱{{ food.price.toFixed(2) }}
              </td>

              <td>
                <span
                  :class="food.available
                    ? 'available'
                    : 'unavailable'"
                >
                  {{ food.available
                    ? 'Available'
                    : 'Unavailable' }}
                </span>
              </td>

              <td class="actions">

                <button
                  class="btn-edit"
                  @click="editFood(food)"
                >
                  Edit
                </button>

                <button
                  class="btn-delete"
                  @click="deleteFood(food.id)"
                >
                  Delete
                </button>

              </td>

            </tr>

          </tbody>

        </table>

      </div>

    </section>

  </div>
</template>