<template lang="pug">
#app
  .jumbotron
    .container.text-left
      transition(name="fade" mode="out-in")
        section(v-if="!showResultTable" key="1")
          form(@submit.prevent='handleSubmit'  @keyup="toggleSubmit")
            .form-group
              label(for='email') Email
              input#email.form-control(
                type='email' 
                v-model='user.email' 
                v-validate="'required|email'" 
                name='email' 
                :class="{ 'is-invalid': submitted && errors.has('email') }")
              .invalid-feedback(v-if="submitted && errors.has('email')")
                | {{ errors.first("email") }}
            
            .form-group
              label(for='firstName') First Name
              input#firstName.form-control(
                type='text' 
                v-model='user.firstName' 
                v-validate="'required'" 
                name='firstName' 
                :class="{ 'is-invalid': submitted && errors.has('firstName') }")
              .invalid-feedback(v-if="submitted && errors.has('firstName')")
                | {{ errors.first("firstName") }}

            .form-group
              label(for='lastName') Last Name
              input#lastName.form-control(
                type='text' 
                v-model='user.lastName' 
                v-validate="'required'" 
                name='lastName' 
                :class="{ 'is-invalid': submitted && errors.has('lastName') }")
              .invalid-feedback(v-if="submitted && errors.has('lastName')")
                | {{ errors.first("lastName") }}
            
            .form-group
              label(for='password') Phone
              input#phone.form-control(
                type='phone' 
                v-model='user.phone' 
                v-validate='{ required: true, digits: 6 }' 
                name='phone' 
                :class="{ 'is-invalid': submitted && errors.has('phone') }")
              .invalid-feedback(v-if="submitted && errors.has('phone')")
                | {{ errors.first("phone") }}
            .text-right.mt-4
              button(@click='addGuest' class="btn btn-outline-primary") Add guests

            div(v-for="(guest, index) in user.guests")
              .form-group
                label(:for="transformLabel(guest.label)" @dblclick="removeGuest") {{ guest.label }}
                input.form-control(
                  type='text' 
                  :name='transformLabel(guest.label)'
                  v-model="guest.value"
                  v-validate='{ required: true, min: 6 }'
                  :class="{ 'is-invalid': submitted && errors.has(transformLabel(guest.label)) }")
                .invalid-feedback(v-if="submitted && errors.has(transformLabel(guest.label))")
                  | {{ errors.first(transformLabel(guest.label)) }}

            .form-group.py-4.text-center
              button.btn(:disabled="disableSubmit" :class="{'btn-primary': !disableSubmit}") Send data

        section(v-else key="2")
          h2.text-center.pb-5.text-success All done !

          table.table
            tbody
              tr(v-for="(title, index) in tableTitles")
                td.py-3 {{ formatTitle(title) }}
                td(v-if="title !== 'guests'") {{ tableValues[index] }}

                template(v-if="title == 'guests'")
                  table.table
                    tbody
                      tr(v-for="item in tableValues[index]")
                        td  {{ item.value  }}
</template>

<script>
import "bootstrap-css";

export default {
  name: "app",
  data: () => ({
    user: {
      firstName: "",
      lastName: "",
      email: "",
      phone: "",
      guests: []
    },
    submitted: false,
    showResultTable: false,
    disableSubmit: true
  }),
  computed: {
    guestsLength() {
      return this.user.guests.length;
    },
    tableTitles() {
      return Object.keys(this.user);
    },
    tableValues() {
      return Object.values(this.user);
    }
  },
  methods: {
    handleSubmit(e) {
      this.submitted = true;
      e.preventDefault();
      this.$validator.validate().then(valid => {
        if (valid) {
          console.log(JSON.stringify(this.user));
          this.showResultTable = true;
        }
      });
    },
    addGuest(e) {
      e.preventDefault();
      this.user.guests.push({
        label: `Guest ${this.guestsLength + 1}`,
        value: ""
      });
    },
    removeGuest(n) {
      this.user.guests.splice(n, 1);
    },
    transformLabel(label) {
      return label.toLowerCase().replace(" ", "-");
    },
    formatTitle(title) {
      return this.unCamelCase(title);
    },
    unCamelCase(str) {
      str = str.replace(/([a-z\xE0-\xFF])([A-Z\xC0\xDF])/g, "$1 $2");
      str = str.toLowerCase();
      return str;
    },
    toggleSubmit() {
      this.disableSubmit = Object.values(this.user).includes("") ? true : false;
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir" Helvetica Arial sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
  background-color: #eceeef;
}

.container {
  padding: 2em;
  max-width: 960px;
}

.table td {
  border-top: 1px dotted #000;
  border-bottom: 1px dotted #000;
  border-left: 1px dotted #000;
}

.table:first-of-type {
  border: 1px dotted #000;
  margin-bottom: 0;
}

.table td {
  vertical-align: middle;
  padding: 1em 2em;
}

.table .table {
  background-color: transparent;
  border: 0;
  margin: 0;
}

label {
  font-weight: 700;
}

td:first-of-type::first-letter {
  text-transform: uppercase;
}

.invalid-feedback {
  display: none;
  margin-top: 0.25rem;
  font-size: 0.875rem;
  color: #dc3545;
}

.custom-select.is-invalid ~ .invalid-feedback,
.custom-select.is-invalid ~ .invalid-tooltip,
.form-control.is-invalid ~ .invalid-feedback,
.form-control.is-invalid ~ .invalid-tooltip,
.was-validated .custom-select:invalid ~ .invalid-feedback,
.was-validated .custom-select:invalid ~ .invalid-tooltip,
.was-validated .form-control:invalid ~ .invalid-feedback,
.was-validated .form-control:invalid ~ .invalid-tooltip {
  display: block;
}

.custom-select.is-invalid,
.form-control.is-invalid,
.was-validated .custom-select:invalid,
.was-validated .form-control:invalid {
  border-color: #dc3545;
}

.custom-select.is-valid,
.form-control.is-valid,
.was-validated .custom-select:valid,
.was-validated .form-control:valid {
  border-color: #28a745;
}

.form-control.is-valid,
.was-validated .form-control:valid {
  border-color: #28a745;
  padding-right: calc(1.5em + 0.75rem);
  background-position: center right calc(0.375em + 0.1875rem);
  background-size: calc(0.75em + 0.375rem) calc(0.75em + 0.375rem);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
