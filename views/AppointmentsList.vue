<template>
  <div>
    <nav class="navbar navbar-light bg-white shadow-sm mb-4">
      <div class="container d-flex justify-content-between align-items-center">
        <span class="fw-bold fs-5">Doctor Appointments</span>
        <router-link to="/" class="btn btn-outline-primary">⇆ Switch Page</router-link>
      </div>
    </nav>

    <div class="container">
      <div class="card shadow-sm">
        <div class="card-header">
          <h5 class="mb-0">Appointments</h5>
        </div>

        <div class="card-body p-0">
          <table class="table table-bordered table-striped mb-0">
            <thead class="table-light">
              <tr>
                <th>Name</th>
                <th>Symptoms</th>
                <th>Time</th>
                <th>Status</th>
                <th>Update</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="appointment in appointments" :key="appointment.appointmentId">
                <td>{{ appointment.patientName }}</td>
                <td>{{ appointment.symptoms }}</td>
                <td>{{ appointment.slot }}</td>
                <td>{{ appointment.status }}</td>
                <td>
                  <select
                    class="form-select"
                    :value="appointment.status"
                    @change="e => updateStatus(appointment, e.target.value)"
                  >
                    <option>Pending</option>
                    <option>In Progress</option>
                    <option>Completed</option>
                  </select>
                </td>
              </tr>

              <tr v-if="appointments.length === 0">
                <td colspan="5" class="text-center p-3">
                  No appointments found.
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const API_BASE_URL = "https://kd5k8ocfs3.execute-api.eu-central-1.amazonaws.com";

export default {
  name: "AppointmentsList",

  data() {
    return {
      appointments: []
    };
  },

  mounted() {
    this.fetchAppointments();
  },

  methods: {
    async fetchAppointments() {
      try {
        const response = await fetch(`${API_BASE_URL}/appointments`);
        const data = await response.json();

        // Handles both normal API Gateway response and Lambda test-style response
        const parsed = data.body ? JSON.parse(data.body) : data;

        this.appointments = parsed;

      } catch (error) {
        console.error("Error fetching appointments:", error);
        alert("Failed to load appointments.");
      }
    },

    async updateStatus(appointment, newStatus) {
      const url = `${API_BASE_URL}/appointments/${appointment.appointmentId}`;

      const payload = {
        status: newStatus
      };

      try {
        const response = await fetch(url, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify(payload)
        });

        const rawBody = await response.text();

        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${rawBody}`);
        }

        alert("Status updated!");

        // Update locally
        appointment.status = newStatus;

      } catch (error) {
        console.error("Failed to update status:", error);
        alert("Update failed. See console for details.");
      }
    }
  }
};
</script>
