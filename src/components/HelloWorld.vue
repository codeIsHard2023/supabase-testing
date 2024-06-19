<script setup>
import { ref, computed } from "vue";
import supabase from "../config/supabaseClient";

defineProps({
  msg: String,
});

const users = ref(null);
const username = ref("");
const email = ref("");
const password = ref("");
const passwordConfirm = ref("");
const errorMessage = ref(null);
const successMessage = ref(null);

const fetchUser = async () => {
  const { data, error } = await supabase.from("user").select();
  if (error) {
    errorMessage.value = `Error ${error} occured on fetch user data `;
    console.error(error);
    users.value = null;
    errorMessage.value = error;
  }
  if (data) {
    users.value = data;
    errorMessage.value = null;
    console.log(users.value);
  }
};

const emailValid = computed(() => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email.value);
});

const passwordValid = computed(() => {
  const passwordRegex =
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
  return passwordRegex.test(password.value);
});

const formValid = computed(() => {
  return (
    emailValid.value &&
    passwordValid.value &&
    password.value === passwordConfirm.value
  );
});

const submitForm = async () => {
  if (!formValid.value) {
    return;
  }

  const newUser = {
    username: username.value,
    email: email.value,
    password: password.value,
  };

  const { error } = await supabase.from("user").insert(newUser);

  if (error) {
    errorMessage.value = `Error: ${error.message}`;
    successMessage.value = null;
  } else {
    errorMessage.value = null;
    successMessage.value = "User created successfully";
    username.value = "";
    email.value = "";
    password.value = "";
    passwordConfirm.value = "";
  }
};

const resetUsers = () => {
  users.value = null;
  errorMessage.value = null;
};
</script>

<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <button type="button" @click="fetchUser">Fetch users</button>
    <button type="button" @click="resetUsers">Reset</button>
    <ul>
      <li v-for="user in users" :key="user.id">
        {{ user.username }}
        {{ user.email }}
      </li>
    </ul>
  </div>
  <form @submit.prevent="submitForm" class="form">
    <div class="form-group">
      <label for="username">Username:</label>
      <input
        type="text"
        id="username"
        v-model="username"
        required
        class="form-input"
      />
    </div>
    <div class="form-group">
      <label for="email">Email:</label>
      <input
        type="email"
        id="email"
        v-model="email"
        required
        class="form-input"
      />
      <span v-if="!emailValid" class="form-error"
        >Please enter a valid email address</span
      >
    </div>
    <div class="form-group">
      <label for="password">Password:</label>
      <input
        type="password"
        id="password"
        v-model="password"
        required
        class="form-input"
      />
      <span v-if="!passwordValid" class="form-error"
        >Password must be at least 8 characters long and contain at least one
        uppercase letter, one lowercase letter, one number, and one special
        character</span
      >
    </div>
    <div class="form-group">
      <label for="password-confirm">Confirm Password:</label>
      <input
        type="password"
        id="password-confirm"
        v-model="passwordConfirm"
        required
        class="form-input"
      />
      <span v-if="password !== passwordConfirm" class="form-error"
        >Passwords do not match</span
      >
    </div>
    <div class="form-group">
      <button type="submit" class="form-button" :disabled="!formValid">
        Submit
      </button>
    </div>
    <div v-if="errorMessage" class="form-error">
      {{ errorMessage }}
    </div>
    <div v-if="successMessage" class="form-success">
      {{ successMessage }}
    </div>
  </form>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
.form {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 300px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 10px;
  width: 100%;
}

.form-input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
}

.form-button {
  padding: 10px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
}

.form-error {
  color: red;
  margin-top: 10px;
}
</style>
