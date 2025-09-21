<script setup lang="ts">
const params = useRoute().params;

const { data: blog } = await useAsyncData(() =>
  queryCollection("content").path(`/blogs/${params.slug}`).first()
);

useSeoMeta({
  title: blog.value?.title,
  description: blog.value?.description,
});

const supabase = useSupabaseClient();
const user = useSupabaseUser();

const email = ref("");
const password = ref("");
const loading = ref(false);
const error = ref("");

const signIn = async () => {
  try {
    loading.value = true;
    error.value = "";

    const { error: signInError } = await supabase.auth.signInWithPassword({
      email: email.value,
      password: password.value,
    });

    if (signInError) {
      error.value = signInError.message;
    }
  } catch (err) {
    error.value = "An unexpected error occurred";
  } finally {
    loading.value = false;
  }
};

const signOut = async () => {
  await supabase.auth.signOut();
};
</script>

<template>
  <ContentRenderer v-if="blog" :value="blog" />
  <div v-else>Blog not found</div>

  <!-- Login Form -->
  <div v-if="!user" class="login-form">
    <h2>Login</h2>
    <form @submit.prevent="signIn">
      <div>
        <label for="email">Email:</label>
        <input
          id="email"
          v-model="email"
          type="email"
          required
          placeholder="Enter your email"
        />
      </div>
      <div>
        <label for="password">Password:</label>
        <input
          id="password"
          v-model="password"
          type="password"
          required
          placeholder="Enter your password"
        />
      </div>
      <button type="submit" :disabled="loading">
        {{ loading ? "Signing in..." : "Sign In" }}
      </button>
    </form>
    <p v-if="error" class="error">{{ error }}</p>
  </div>

  <!-- User Info -->
  <div v-else class="user-info">
    <h2>Welcome!</h2>
    <p class="success">Successfully logged in!</p>
    <p><strong>User ID:</strong> {{ user.id }}</p>
    <p><strong>Email:</strong> {{ user.email }}</p>
    <button @click="signOut" class="sign-out-btn">Sign Out</button>
  </div>
</template>

<style scoped>
.login-form,
.user-info {
  max-width: 400px;
  margin: 20px auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: #f9f9f9;
}

.login-form div {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

button {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  width: 100%;
}

button:hover:not(:disabled) {
  background-color: #0056b3;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.sign-out-btn {
  background-color: #dc3545;
  margin-top: 10px;
}

.sign-out-btn:hover {
  background-color: #c82333;
}

.error {
  color: #dc3545;
  margin-top: 10px;
}

.success {
  color: #28a745;
  font-weight: bold;
}

.user-info p {
  margin: 10px 0;
}
</style>
