<template>
  <div class="wrapper">
    <h1 class="title">Login Failed</h1>
    <div class="image-container">
      <img src="@/assets/unauthorized.jpg" alt="Email error" class="error-image">
    </div>
    <div class="text-container">
      <p class="message">The email address <span class="email-highlight">{{ attemptedEmail }}</span> is not registered.</p>
      <p class="hint">Please check your email or sign up for a new account.</p>
      
    </div>
    <div class="button-group">
      <button @click="goToSignup" class="action-button signup-button">Sign Up Now</button>
      
    </div>
  </div>
</template>

<script>
export default {
  name: 'LoginErrorPage',
  props: {
    attemptedEmail: {
      type: String,
      default: "example@domain.com"
    }
  },
  methods: {
    goToSignup() {
      this.$router.push('/signup');
    },
    goToLogin() {
      this.$router.push('/login');
    }
  }
};
</script>

<style scoped>
body {
  background: #f8f9fa;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  margin: 0;
  font-family: 'Inter', sans-serif;
  background-image: linear-gradient(135deg, #f5f7fa 0%, #e4f0f8 100%);
}

.wrapper {
  width: 100%;
  max-width: 500px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 16px;
  padding: 40px;
  background: white;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  text-align: center;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.title {
  font-size: 28px;
  font-weight: 700;
  color: #dc3545;
  margin-bottom: 24px;
  position: relative;
  padding-bottom: 12px;
}

.title::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 4px;
  background: #dc3545;
  border-radius: 2px;
}

.image-container {
  margin: 0 auto 24px;
  width: 180px;
  height: 180px;
}

.error-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.text-container {
  margin-bottom: 32px;
  line-height: 1.6;
}

.message {
  font-size: 18px;
  color: #4a5568;
  margin-bottom: 12px;
  font-weight: 500;
}

.email-highlight {
  font-weight: 600;
  color: #dc3545;
  background: #fee;
  padding: 2px 6px;
  border-radius: 4px;
}

.hint {
  font-size: 16px;
  color: #6c757d;
  margin-bottom: 8px;
}

.suggestion {
  font-size: 15px;
  color: #6c757d;
  font-style: italic;
  margin-bottom: 0;
}

.button-group {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 20px;
}

.action-button {
  padding: 12px 24px;
  font-size: 16px;
  font-weight: 600;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  width: 100%;
  border: none;
}

.signup-button {
  background: #28a745;
  color: white;
  box-shadow: 0 4px 6px rgba(40, 167, 69, 0.1);
}

.signup-button:hover {
  background: #218838;
  transform: translateY(-2px);
  box-shadow: 0 6px 8px rgba(40, 167, 69, 0.15);
}

.login-button {
  background: white;
  color: #495057;
  border: 1px solid #dee2e6;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.login-button:hover {
  background: #f8f9fa;
  border-color: #adb5bd;
  transform: translateY(-2px);
}

@media (min-width: 480px) {
  .button-group {
    flex-direction: row;
    justify-content: center;
  }
  
  .action-button {
    width: auto;
    min-width: 180px;
  }
}
</style>