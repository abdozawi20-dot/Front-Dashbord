<script setup lang="ts">
import { ref, onMounted } from 'vue'

interface Facility {
  id: number
  name: string
  type: string
  location: string
  capacity: number
  description?: string
  image_url?: string
}

interface Reservation {
  id: number
  user_id: number
  facility_id: number
  start_time: string
  end_time: string
  status: string
  purpose: string
}

interface User {
  id: number
  name: string
  email: string
  role: string
}

const facilities = ref<Facility[]>([])
const reservations = ref<Reservation[]>([])
const users = ref<User[]>([])

const loading = ref(true)
const error = ref<string | null>(null)

const fetchData = async () => {
  try {
    const [facRes, resRes, userRes] = await Promise.all([
      fetch('http://localhost:3000/api/admin/facilities'),
      fetch('http://localhost:3000/api/admin/reservations'),
      fetch('http://localhost:3000/api/user')
    ])

    if (!facRes.ok || !resRes.ok || !userRes.ok) {
      throw new Error('Failed to fetch some resources')
    }

    facilities.value = await facRes.json()
    reservations.value = await resRes.json()
    users.value = await userRes.json()
  } catch (e: any) {
    error.value = e.message
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchData()
})
</script>

<template>
  <div class="dashboard">
    <header>
      <div class="wrapper">
        <h1>Resource Management Dashboard</h1>
      </div>
    </header>

    <main>
      <div v-if="loading" class="loading">Loading dashboard data...</div>
      <div v-else-if="error" class="error">Error: {{ error }}</div>
      <div v-else class="content">
        
        <section class="section">
          <h2>Facilities</h2>
          <div class="facilities-grid">
            <div v-for="facility in facilities" :key="facility.id" class="card facility-card">
              <img v-if="facility.image_url" :src="facility.image_url" :alt="facility.name" class="facility-image" />
              <div class="card-body">
                <h3>{{ facility.name }}</h3>
                <p class="type">{{ facility.type }}</p>
                <p class="description">{{ facility.description }}</p>
                <div class="details">
                  <span>📍 {{ facility.location }}</span>
                  <span>👥 {{ facility.capacity }}</span>
                </div>
              </div>
            </div>
          </div>
        </section>

        <div class="row">
          <section class="section half">
            <h2>Recent Reservations</h2>
            <div class="table-container">
              <table>
                <thead>
                  <tr>
                    <th>ID</th>
                    <th>Purpose</th>
                    <th>Status</th>
                    <th>Time</th>
                  </tr>
                </thead>
                <tbody>
                  <div v-if="reservations.length === 0" class="empty-state">No reservations found.</div>
                  <tr v-for="res in reservations" :key="res.id">
                    <td>#{{ res.id }}</td>
                    <td>{{ res.purpose }}</td>
                    <td><span :class="['status', res.status]">{{ res.status }}</span></td>
                    <td>{{ new Date(res.start_time).toLocaleDateString() }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </section>

          <section class="section half">
            <h2>Users</h2>
            <div class="table-container">
              <table>
                <thead>
                  <tr>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Role</th>
                  </tr>
                </thead>
                <tbody>
                   <div v-if="users.length === 0" class="empty-state">No users found.</div>
                  <tr v-for="user in users" :key="user.id">
                    <td>{{ user.name }}</td>
                    <td>{{ user.email }}</td>
                    <td><span :class="['role', user.role]">{{ user.role }}</span></td>
                  </tr>
                </tbody>
              </table>
            </div>
          </section>
        </div>

      </div>
    </main>
  </div>
</template>

<style scoped>
.dashboard {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  font-family: 'Inter', sans-serif;
  color: #333;
}

header {
  margin-bottom: 3rem;
  border-bottom: 1px solid #eee;
  padding-bottom: 1rem;
}

h1 {
  font-weight: 800;
  font-size: 2rem;
  color: #2c3e50;
}

h2 {
  font-size: 1.5rem;
  margin-bottom: 1.5rem;
  color: #2c3e50;
  border-left: 4px solid #42b983;
  padding-left: 1rem;
}

.section {
  margin-bottom: 3rem;
}

.row {
  display: flex;
  gap: 2rem;
  flex-wrap: wrap;
}

.half {
  flex: 1;
  min-width: 300px;
}

/* Facilities Grid */
.facilities-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 2rem;
}

.card {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
  overflow: hidden;
  transition: transform 0.2s;
  border: 1px solid #eee;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 15px rgba(0,0,0,0.1);
}

.facility-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.card-body {
  padding: 1.5rem;
}

.card-body h3 {
  margin: 0 0 0.5rem 0;
  font-size: 1.25rem;
}

.type {
  color: #666;
  font-size: 0.9rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 1rem;
}

.description {
  font-size: 0.95rem;
  line-height: 1.5;
  margin-bottom: 1.5rem;
  color: #444;
}

.details {
  display: flex;
  justify-content: space-between;
  font-size: 0.9rem;
  color: #666;
  border-top: 1px solid #eee;
  padding-top: 1rem;
}

/* Tables */
.table-container {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
  overflow: hidden;
  border: 1px solid #eee;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 1rem;
  text-align: left;
  border-bottom: 1px solid #eee;
}

th {
  background: #f8f9fa;
  font-weight: 600;
  color: #666;
}

tr:last-child td {
  border-bottom: none;
}

/* Status Badges */
.status {
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.85rem;
  font-weight: 500;
}

.status.confirmed {
  background: #e6fffa;
  color: #047857;
}

.status.pending {
  background: #fffbeb;
  color: #b45309;
}

.status.cancelled {
  background: #fef2f2;
  color: #b91c1c;
}

.role {
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.85rem;
  font-weight: bold;
  text-transform: uppercase;
}

.role.admin {
  background: #e0e7ff;
  color: #3730a3;
}

.role.user {
  background: #f3f4f6;
  color: #374151;
}

.loading, .error, .empty-state {
  text-align: center;
  padding: 2rem;
  font-size: 1.1rem;
  color: #666;
}

.error {
  color: #dc2626;
}
</style>
