<template>
  <div>
    <div class="banner">
      <div class="logo-container">
        <img :src="getImageUrl('sjp-logo.jpg')" alt="University Logo" class="logo" />
      </div>
      <div class="title">
        <h1>Japura Net Forum</h1>
        <p>Registration</p>
      </div>
    </div>

    <form @submit.prevent="handleSignup">
      <label for="firstName">First Name</label>
      <input type="text" id="firstName" v-model="firstName" placeholder="Enter your first name" required>

      <label for="lastName">Last Name</label>
      <input type="text" id="lastName" v-model="lastName" placeholder="Enter your last name" required>

      <label for="email">Email</label>
      <input type="email" id="email" v-model="email" placeholder="Enter your university email" required>

      <label for="faculty">Faculty of Studying</label>
      <select id="faculty" v-model="faculty" required>
        <option value="" disabled selected>Select your faculty</option>
        <option v-for="fac in faculties" :key="fac" :value="fac">{{ fac }}</option>
      </select>

      <label for="password">Password</label>
      <input type="password" id="password" v-model="password" placeholder="Create a password" required>

      <label for="confirmPassword">Confirm Password</label>
      <input type="password" id="confirmPassword" v-model="confirmPassword" placeholder="Confirm your password"
        required>
      <div v-if="passwordError" class="error">{{ passwordError }}</div>

      <div class="signup">
        <button type="submit">Signup</button>
      </div>

      <div class="login-section">
        <p>Already have an account?</p>
        <a @click="goToLogin" href="#">Login</a>
      </div>
    </form>
  </div>
</template>

<!-- <script setup>
import { ref } from 'vue';
import { getAuth, createUserWithEmailAndPassword } from 'firebase/auth';
import { useRouter } from 'vue-router';

const email = ref('');
const password = ref('');
const router = useRouter(); 

const register = () => {
  const auth = getAuth()
  createUserWithEmailAndPassword(auth, email.value, password.value)
    .then((data) => {
      console.log("Registered successfully!");
      console.log(auth.currentUser);
      router.push('/home'); // Redirect to home page after successful registration
      })
      .catch((error) => {
        console.error(error.code);
        alert(error.code);
        });
        }
        
        </script> -->

<script>
import { getAuth, createUserWithEmailAndPassword } from 'firebase/auth';

export default {
  name: 'SignupForm',
  data() {
    return {
      firstName: '',
      lastName: '',
      email: '',
      faculty: '',
      password: '',
      confirmPassword: '',
      passwordError: '',
      faculties: [
        'Humanities and Social Sciences',
        'Applied Sciences',
        'Management Studies and Commerce',
        'Medical Sciences',
        'Graduate Studies',
        'Technology',
        'Engineering',
        'Allied Health Sciences',
        'Dental Sciences',
        'Urban and Aquatic Bioresources',
        'Computing',
      ],
      getImageUrl(imageName) {
        return require(`@/assets/${imageName}`);
      },
    }
  },
  methods: {
    handleSignup() {
      // Reset error message
      this.passwordError = '';

      // Validate passwords match
      if (this.password !== this.confirmPassword) {
        this.passwordError = 'Passwords do not match';
        return;
      }

      // Check if email has a valid university domain (example validation)
      if (!this.email.includes('.sjp') && !this.email.includes('.ac')) {
        this.passwordError = 'Please use a valid university email address';
        return;
      }

      // Form data to be sent to API
      const userData = {
        firstName: this.firstName,
        lastName: this.lastName,
        email: this.email,
        faculty: this.faculty,
        password: this.password
      };
      
      // Firebase authentication
      const auth = getAuth();
      createUserWithEmailAndPassword(auth, userData.email, userData.password)
        .then((data) => {
          console.log("Registered successfully!");
          console.log(auth.currentUser);

          // Get JWT token
          return auth.currentUser.getIdToken();
        })
        .then((idToken) => {
          console.log("Firebase ID Token:", idToken);

          // Send userData and idToken to backend via graphql
          const mutation = `
            mutation {
              registerUser(
                firstName: "${userData.firstName}",
                lastName: "${userData.lastName}",
                email: "${userData.email}",
                faculty: "${userData.faculty}",
                password: "${userData.password}",
                idToken: "${idToken}"
              ) {
                  id
                  email
                }
            }
          `;

          return fetch('https://graphql-endpoint-url', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${idToken}`
            },
            body: JSON.stringify({ query: mutation })
          });
        })
        .then((res => res.json()))
        .then(result => {
          console.log('Backend response:', result);
          alert('Account created successfully! Redirecting to home page...');

          // Redirect to home page after successful signup
          this.$router.push('/home');
        })
        .catch((error) => {
          console.error(error.code);
          alert(error.code);
        });

      console.log('Signup form submitted:', userData);

    },
    goToLogin() {
      this.$emit('switch-to-login');
    }
  }
}
</script>

<style scoped>
form {
  max-width: 420px;
  margin: 30px auto;
  background: white;
  text-align: left;
  padding: 40px;
  border-radius: 10px;
}

label {
  color: #aaa;
  display: inline-block;
  margin: 25px 0 15px;
  font-size: 0.8em;
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: bold;
}

input,
select {
  display: block;
  padding: 10px 6px;
  width: 100%;
  box-sizing: border-box;
  border: none;
  border-bottom: 1px solid #ddd;
  color: #555;
}

.banner {
  display: flex;
  max-width: 420px;
  margin: 10px auto;
  align-items: flex-start;
  height: 80px;
}

.title {
  display: flex;
  flex-direction: column;
  padding-left: 5px;
  justify-content: center;
  align-items: flex-start;
  border-left-width: 2px;
  border-left-color: #ddd;
  border-left-style: solid;
}

.title h1 {
  margin: 0;
  color: #555;
}

.title p {
  font-size: small;
  text-transform: uppercase;
  color: #aaa;
}

.logo-container {
  padding: 0 5px 5px 0;
  max-width: 80px;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
}

.logo {
  width: 100%;
  object-fit: cover;
}

button {
  background: #c3262d;
  border: 0;
  padding: 10px 20px;
  margin-top: 20px;
  color: white;
  border-radius: 20px;
  cursor: pointer;
}

.signup {
  text-align: center;
}

.login-section {
  margin-top: 30px;
  text-align: left;
}

.login-section p {
  color: #555;
}

.login-section a {
  color: #c3262d;
  text-decoration: none;
  font-weight: bold;
}

.error {
  color: #ff0062;
  margin-top: 10px;
  font-size: 0.8em;
  font-weight: bold;
}
</style>