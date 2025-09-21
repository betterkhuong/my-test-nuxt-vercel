<template>
  <div>
    <h1>App page</h1>

    <!-- Login Form -->
    <div v-if="!user" class="login-form">
      <h2>Login with OTP</h2>

      <!-- Email Input Step -->
      <div v-if="!otpSent">
        <form @submit.prevent="sendOTP">
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
          <button type="submit" :disabled="loading">
            {{ loading ? "Sending OTP..." : "Send OTP" }}
          </button>
        </form>
      </div>

      <!-- OTP Verification Step -->
      <div v-else>
        <p class="otp-sent">OTP sent to {{ email }}</p>
        <form @submit.prevent="verifyOTP">
          <div>
            <label for="otp">Enter OTP:</label>
            <input
              id="otp"
              v-model="otp"
              type="text"
              required
              placeholder="Enter 6-digit OTP"
              maxlength="6"
            />
          </div>
          <button type="submit" :disabled="loading">
            {{ loading ? "Verifying..." : "Verify OTP" }}
          </button>
        </form>
        <button @click="resetForm" class="back-btn">Back to Email</button>
      </div>

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
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const supabase = useSupabaseClient();
const user = useSupabaseUser();

const email = ref("");
const otp = ref("");
const loading = ref(false);
const error = ref("");
const otpSent = ref(false);

const sendOTP = async () => {
  try {
    loading.value = true;
    error.value = "";

    const { error: otpError } = await supabase.auth.signInWithOtp({
      email: email.value,
    });

    if (otpError) {
      error.value = otpError.message;
    } else {
      otpSent.value = true;
    }
  } catch (err) {
    error.value = "An unexpected error occurred";
  } finally {
    loading.value = false;
  }
};

const verifyOTP = async () => {
  try {
    loading.value = true;
    error.value = "";

    const { error: verifyError } = await supabase.auth.verifyOtp({
      email: email.value,
      token: otp.value,
      type: "email",
    });

    if (verifyError) {
      error.value = verifyError.message;
    }
  } catch (err) {
    error.value = "An unexpected error occurred";
  } finally {
    loading.value = false;
  }
};

const resetForm = () => {
  otpSent.value = false;
  otp.value = "";
  error.value = "";
};

const signOut = async () => {
  await supabase.auth.signOut();
};
</script>

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

.otp-sent {
  color: #28a745;
  font-weight: bold;
  margin-bottom: 15px;
}

.back-btn {
  background-color: #6c757d;
  margin-top: 10px;
}

.back-btn:hover {
  background-color: #5a6268;
}
</style>
