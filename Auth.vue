<script lang="ts">
import { ref, Ref } from "vue";

export default {
  name: "AuthApp",
  setup() {
    // Reactive state with TypeScript annotations
    const credentials: Ref<{ username: string; password: string }> = ref({
      username: "",
      password: "",
    });

    const token: Ref<string | null> = ref(localStorage.getItem("authToken"));
    const protectedData: Ref<string | null> = ref(null);

    // Login handler
    const login = async (): Promise<void> => {
      try {
        const response = await fetch("https://example.com/api/login", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(credentials.value),
        });

        if (!response.ok) throw new Error("Login failed");
        const data = await response.json();

        // Store token securely (not the best practice; discuss HttpOnly cookies)
        localStorage.setItem("authToken", data.token);
        token.value = data.token;
        alert("Login successful!");
      } catch (error) {
        console.error("Error during login:", error);
        alert("An error occurred during login.");
      }
    };

    // Fetch protected data handler
    const fetchProtectedData = async (): Promise<void> => {
      try {
        if (!token.value) {
          alert("You are not logged in!");
          return;
        }

        const response = await fetch("https://example.com/api/protected", {
          headers: {
            Authorization: `Bearer ${token.value}`,
          },
        });

        if (!response.ok) throw new Error("Failed to fetch protected data");
        const data = await response.json();
        protectedData.value = data.message;
      } catch (error) {
        console.error("Error fetching protected data:", error);
        alert("Failed to fetch protected data.");
      }
    };

    return {
      credentials,
      token,
      protectedData,
      login,
      fetchProtectedData,
    };
  },
};
</script>

<template>
  <div class="auth-app">
    <h1>Login</h1>
    <form @submit.prevent="login">
      <label for="username">Username:</label>
      <input
        type="text"
        id="username"
        v-model="credentials.username"
        placeholder="Enter username"
      />

      <label for="password">Password:</label>
      <input
        type="password"
        id="password"
        v-model="credentials.password"
        placeholder="Enter password"
      />

      <button type="submit">Login</button>
    </form>

    <button v-if="token" @click="fetchProtectedData">Get Protected Data</button>
    <div v-if="protectedData">
      <h2>Protected Data</h2>
      <p>{{ protectedData }}</p>
    </div>
  </div>
</template>

<style scoped>
.auth-app {
  font-family: Arial, sans-serif;
  max-width: 400px;
  margin: auto;
}
form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
button {
  margin-top: 10px;
}
</style>

