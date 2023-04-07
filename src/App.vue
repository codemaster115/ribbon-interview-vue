<template>
  <v-app>
    <v-app-bar>
      <v-container class="d-flex align-center py-0">
        <v-app-bar-title class="pl-0">
          <div class="d-flex align-center">Ribbon</div>
        </v-app-bar-title>
      </v-container>
    </v-app-bar>

    <v-main>
      <section id="hero">
        <v-sheet class="d-flex align-center pb-16" color="grey-darken-3">
          <v-container class="text-center">
            <v-responsive class="mx-auto">
              <h3 class="text-h3">Try Ribbon's all new features</h3>

              <p class="mt-4 text-medium-emphasis">
                Our all-in-one platform gives you the banking, accounting, fundraising, and organizational tools you
                need to build a successful charity under the umbrella of your fiscal sponsor.
              </p>
            </v-responsive>
          </v-container>
        </v-sheet>
      </section>

      <v-sheet>
        <section id="filter">
          <v-container>
            <v-row justify="space-between">
              <v-col cols="auto">
                <h2 class="text-h4">Ribbon Donor List</h2>
                <p class="text-primary mt-3">In Beta now!</p>
                <p class="mt-3">See all those that have given in one place!</p>
              </v-col>
            </v-row>
            <v-card>
              <v-row>
                <v-col cols="3" style="margin: 10px">
                  <v-text-field
                    v-model="filterKey"
                    append-icon="mdi-magnify"
                    label="Search"
                    single-line
                    hide-details
                    density="compact"
                    outlined
                  ></v-text-field>
                </v-col>
              </v-row>
              <v-data-table
                v-resize="onResize"
                :headers="donorHeader"
                :items="donors"
                fixed-header
                :height="windowSize.y - 450"
                :search="filterKey"
                item-key="id"
                sort-by="LastModified"
                :sort-desc="true"
                must-sort
                loading-text="Loading... Please wait"
                :items-per-page="5"
                @click:row="handleClick"
              >
              </v-data-table>
            </v-card>
          </v-container>
        </section>
      </v-sheet>

      <v-sheet class="py-16" color="#1818181a">
        <section id="grid">
          <v-container>
            <v-row justify="space-between">
              <v-col cols="auto">
                <v-responsive width="350">
                  <h2 class="text-h4">Show your support feature</h2>
                  <p class="text-success mt-3">Available now!</p>
                  <p class="mt-3">Easily send messages to those that have given!</p>
                </v-responsive>
              </v-col>
              <v-col cols="5" style="background: transparent; border: none">
                <v-form ref="msgForm" v-model="valid" validate-on="submit" @submit.prevent="submit">
                  <v-textarea
                    v-model="message"
                    :rules="messageRules"
                    label="Message"
                    single-line
                    density="compact"
                    outlined
                  ></v-textarea>
                  <v-text-field
                    v-model="email"
                    :rules="emailRules"
                    label="Email"
                    single-line
                    density="compact"
                    outlined
                  ></v-text-field>
                  <v-text-field
                    single-line
                    density="compact"
                    outlined
                    v-model="donor_id"
                    label="Donor Id"
                  ></v-text-field>
                  <v-btn type="submit" block class="mt-2">Send</v-btn>
                </v-form>
              </v-col>
            </v-row>
          </v-container>
        </section>
      </v-sheet>
    </v-main>
    <v-footer>
      <v-container class="text-overline d-flex align-center justify-space-between">
        <div>Copyright &copy; 2023 Flourish Change Inc dba Ribbon</div>

        <v-icon icon="mdi-bank" size="x-large" />
      </v-container>
    </v-footer>
    <v-dialog v-model="isLoading" persistent width="300">
      <v-card color="#00754A" dark>
        <v-card-text>
          Please wait...
          <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
    <v-snackbar v-model="snackbar" :timeout="2000" right top>
      {{ errorMessage }}
      <template v-slot:actions>
        <v-btn color="blue" variant="text" @click="snackbar = false"> Close </v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script>
import axios from 'axios'
export default {
  name: 'App',

  data() {
    return {
      donors: [],
      filterKey: '',
      donorHeader: [
        {
          text: 'Name',
          sortable: true,
          value: 'full_name',
          class: 'donor_header',
        },
        {
          text: 'DONATION AMOUNTS',
          sortable: true,
          value: 'total_donations',
          class: 'donor_header',
        },
        {
          text: 'CREATED AT',
          sortable: true,
          value: 'first_donation',
          class: 'donor_header',
        },
        {
          text: 'EMAIL',
          sortable: true,
          value: 'email',
          class: 'donor_header',
        },
      ],
      isLoading: false,
      windowSize: {
        x: 0,
        y: 0,
      },
      valid: false,
      email: '',
      message: '',
      donor_id: '',
      snackbar: false,
      errorMessage: '',
      emailRules: [
        (value) => {
          if (value.length > 0) return true
          return 'E-mail is required.'
        },
      ],
      messageRules: [
        (value) => {
          if (value && value.length > 15) return true
          return 'Message should be longer than 15 characters.'
        },
      ],
    }
  },
  mounted() {
    this.isLoading = true
    axios
      .get('https://interview.ribbon.giving/api/donors')
      .then((response) => {
        this.donors = response.data.data
        this.isLoading = false
      })
      .catch((err) => {
        this.errorMessage = err.message
        this.snackbar = true
        this.isLoading = false
      })
  },
  methods: {
    async submit() {
      // Send message to server.
      if (this.valid) {
        this.isLoading = true
        axios
          .post(
            `https://interview.ribbon.giving/api/donors/${this.donor_id}/send-message`,
            {
              message: this.message,
              email: this.email,
            },
            {
              headers: {
                'Content-Type': 'application/json',
              },
            }
          )
          .then(() => {
            this.errorMessage = 'Message sent successfully'
          })
          .catch((err) => {
            this.errorMessage = err.message
          })
          .finally(() => {
            this.$refs.msgForm.reset();
            this.isLoading = false
            this.snackbar = true
          })
      }
    },
    handleClick(value) {
      alert(value.id)
    },
    onResize() {
      this.windowSize = { x: window.innerWidth, y: window.innerHeight }
    },
  },
}
</script>
<style>
body {
  color: rgba(58, 58, 64, 0.87) !important;
}
.donor_header {
  background-color: rgba(58, 58, 64, 0.05) !important;
}
tr:hover {
  background-color: rgba(58, 58, 64, 0.03) !important;
}
</style>
