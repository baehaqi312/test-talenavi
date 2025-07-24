<script setup>
import { ref, onMounted, computed } from 'vue'

const grouped = ref({})
const search = ref('')

onMounted(async () => {
  try {
    const res = await fetch('https://mocki.io/v1/282222c9-43cf-4d92-9ba0-0e0d1447f403')
    const data = await res.json()

    const result = data.data.reduce((acc, item) => {
      const key = item.status || 'unknown'
      if (!acc[key]) acc[key] = []
      acc[key].push(item)
      return acc
    }, {})

    grouped.value = result
  } catch (err) {
    console.error('Fetch error:', err)
  }
})

const statusColors = {
  'In Progress': '#1976D2',
  'Ready to start': '#43A047',
  'Waiting for review': '#FBC02D',
  'Stuck': '#E53935',
  'Done': '#7CB342',
  'Pending Deploy': '#00897B',
}

const filteredGrouped = computed(() => {
  const query = search.value.toLowerCase()

  const filtered = {}

  for (const status in grouped.value) {
    const items = grouped.value[status].filter(item =>
      (item.title || '').toLowerCase().includes(query)
    )

    if (items.length > 0) {
      filtered[status] = items
    }
  }

  return filtered
})

const getAvatar = (name) => `https://ui-avatars.com/api/?name=${name}&background=random&size=32&length=1`
</script>

<template>
  <div class="container-fluid">
    <input type="text" v-model="search" placeholder="Search ..." class="form-control my-3" />
    <div class="row">
      <div class="col-12 col-md-6 col-lg-4 col-xl-2 mb-3" v-for="(items, status) in filteredGrouped" :key="status">
        <div class="card h-100 p-0">
          <div class="card-header text-white" :style="{ backgroundColor: statusColors[items[0].status] || '#9E9E9E' }">
            <h5 class="mb-0">{{ status }} {{ items.length }}</h5>
          </div>
          <div class="card-body bg-secondary bg-opacity-10">
            <div v-for="item in items" :key="item.id" class="mb-2">
              <div class="card">
                <div class="card-body">
                  <p class="fw-semibold mb-1">{{ item.title }}</p>
                  <p class="mb-1">{{ item.status }}</p>
                  <div class="bg-secondary bg-opacity-10 border-start border-danger border-4 rounded-end px-2">
                    {{ item.type }}
                  </div>
                  <div class="d-flex align-items-center mt-3">
                    <img v-for="(dev, idx) in item.developer.split(',')" :key="idx" size="32"
                      class="rounded-circle me-1" :src="getAvatar(dev)" alt="Avatar" />
                    <span>{{ item.assignee }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
