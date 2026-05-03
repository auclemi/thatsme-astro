<script setup>
import ErrorComponent from "../error/error.component.vue";
import { ref, computed } from "vue";

const name = ref("");
const email = ref("");
const message = ref("");

const loading = ref(false);
const success = ref(false);
const serverError = ref(null);

// Champs touchés individuellement (blur)
const touched = ref({
  name: false,
  email: false,
  message: false,
});

// Formulaire soumis au moins une fois
const submitted = ref(false);

// Validation
const errors = computed(() => {
  const e = {};

  if (name.value.trim().length < 5) e.name = "Veuillez entrer votre nom.";
  if (!email.value.match(/^[^@\s]+@[^@\s]+\.[^@\s]+$/))
    e.email = "Veuillez entrer un email valide.";
  if (message.value.trim().length < 5)
    e.message = "Veuillez écrire un message.";

  return e;
});

const isValid = computed(() => Object.keys(errors.value).length === 0);

// Submit
async function submitForm(e) {
  e.preventDefault();
  submitted.value = true;

  // Si invalide → on n’envoie pas
  if (!isValid.value) return;

  loading.value = true;
  success.value = false;
  serverError.value = null;
  const contactEndPoint = import.meta.env.PUBLIC_CONTACT_API_URL;
  if (!contactEndPoint) {
    serverError.value =
      "Configuration manquante : définir PUBLIC_CONTACT_API_URL dans .env";
    loading.value = false;
    return;
  }
  try {
    const res = await fetch(contactEndPoint, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        name: name.value,
        email: email.value,
        message: message.value,
      }),
    });

    if (!res.ok) throw new Error("Erreur serveur");

    success.value = true;

    // Reset des champs
    name.value = "";
    email.value = "";
    message.value = "";

    // Reset des états
    touched.value = { name: false, email: false, message: false };
    submitted.value = false;
  } catch (err) {
    serverError.value = err.message;
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <ErrorComponent
    v-if="serverError"
    errorMessage="Une erreur est survenue."
    instruction="Veuillez réessayer dans quelques minutes."
  />
  <div class="alert alert-success" v-if="success">
    <div>Votre message a bien été envoyé.</div>
    <div>Je vous réponds généralement sous 24 à 48h en semaine.</div>
    <div><a href="/home">Retour à l'accueil.</a></div>
  </div>
  <div
    class="row g-5 justify-content-md-between"
    v-if="!serverError && !success"
  >
    <div class="col-12 col-md-5">
      <form @submit="submitForm">
        <!-- Nom -->
        <div>
          <label for="name" class="form-label">Nom :</label>
          <input
            id="name"
            v-model="name"
            type="text"
            class="form-control"
            @blur="touched.name = true"
            :class="{
              'is-invalid': (touched.name || submitted) && errors.name,
            }"
          />
          <div class="invalid-feedback">{{ errors.name }}</div>
        </div>

        <!-- Email -->
        <div class="mt-3">
          <label for="email" class="form-label">Email :</label>
          <input
            id="email"
            v-model="email"
            type="email"
            class="form-control"
            @blur="touched.email = true"
            :class="{
              'is-invalid': (touched.email || submitted) && errors.email,
            }"
          />
          <div class="invalid-feedback">{{ errors.email }}</div>
        </div>

        <!-- Message -->
        <div class="mt-3">
          <label for="message" class="form-label">Message :</label>
          <textarea
            id="message"
            v-model="message"
            class="form-control"
            minlength="5"
            rows="5"
            @blur="touched.message = true"
            :class="{
              'is-invalid': (touched.message || submitted) && errors.message,
            }"
          ></textarea>
          <div class="invalid-feedback">{{ errors.message }}</div>
        </div>

        <!-- Alerte globale -->
        <div
          class="alert alert-warning mt-3 visually-hidden"
          role="alert"
          aria-live="assertive"
          v-if="submitted && !isValid"
        >
          Veuillez corriger les erreurs du formulaire avant de soumettre.
        </div>

        <!-- Submit -->
        <div class="mt-3">
          <button class="btn btn-success" type="submit" :disabled="loading">
            <span
              v-if="loading"
              class="spinner-border spinner-border-sm me-2"
            ></span>
            Envoyer
          </button>
        </div>
      </form>
    </div>

    <!-- Informations légales -->
    <div class="col-12 col-md-4 alert alert-success">
      <h2>Informations légales</h2>
      <div>Michel AUCLERT – Consultant web indépendant</div>
      <div>Perpignan (France)</div>
      <div>SIRET : 31933539400064</div>
    </div>
  </div>
</template>
