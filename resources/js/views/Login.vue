<template>
  <div class="m-auto w-full max-w-[440px]">
    <div class="glass p-10 rounded-[2.5rem] shadow-2xl relative overflow-hidden group">
      <!-- Glow effect -->
      <div class="absolute -top-24 -left-24 w-48 h-48 bg-indigo-500/20 blur-[60px] rounded-full group-hover:bg-indigo-500/30 transition-colors duration-700"></div>
      
      <div class="relative z-10">
        <div class="w-16 h-16 bg-gradient-to-tr from-indigo-500 to-purple-600 rounded-2xl flex items-center justify-center mb-8 mx-auto shadow-lg shadow-indigo-500/20 rotate-3 transition-transform hover:rotate-0 duration-500">
          <svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 00-2 2zm10-10V7a4 4 0 00-8 0v4h8z"></path></svg>
        </div>
        
        <h2 class="text-3xl font-black text-center text-white mb-2 tracking-tight">Content de vous revoir</h2>
        <p class="text-slate-400 text-center text-sm font-medium mb-10">Connectez-vous pour commencer à discuter</p>
        
        <form @submit.prevent="login" class="space-y-6">
          <div class="space-y-2">
            <label class="block text-[13px] font-bold text-slate-400 uppercase tracking-widest px-1">Email professionnel</label>
            <input 
              v-model="form.email"
              type="email" 
              required 
              placeholder="votre@email.com"
              class="w-full px-5 py-4 bg-white/5 border border-white/10 rounded-2xl text-white placeholder-slate-500 outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all font-medium"
            />
          </div>
          
          <div class="space-y-2">
            <label class="block text-[13px] font-bold text-slate-400 uppercase tracking-widest px-1">Mot de passe</label>
            <input 
              v-model="form.password"
              type="password" 
              required 
              placeholder="••••••••"
              class="w-full px-5 py-4 bg-white/5 border border-white/10 rounded-2xl text-white placeholder-slate-500 outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all font-medium"
            />
          </div>
          
          <div v-if="error" class="p-4 bg-red-500/10 border border-red-500/20 rounded-2xl text-red-400 text-sm font-bold text-center animate-shake">
            {{ error }}
          </div>

          <button 
            type="submit" 
            :disabled="loading"
            class="w-full bg-gradient-to-tr from-indigo-500 to-purple-600 hover:from-indigo-600 hover:to-purple-700 text-white font-bold py-4 rounded-2xl transition-all shadow-xl shadow-indigo-600/20 active:scale-[0.98] disabled:opacity-50 disabled:cursor-not-allowed group"
          >
            <div class="flex items-center justify-center gap-2">
              <span v-if="loading" class="w-5 h-5 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
              <span>{{ loading ? 'Authentification...' : 'Se connecter' }}</span>
              <svg v-if="!loading" class="w-4 h-4 transform group-hover:translate-x-1 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7l5 5m0 0l-5 5m5-5H6"></path></svg>
            </div>
          </button>
        </form>
        
        <div class="mt-8 text-center text-[14px] text-slate-400 font-medium">
          Pas encore de compte ? 
          <router-link to="/register" class="text-indigo-400 font-bold hover:text-indigo-300 transition-colors px-1">Créer un profil</router-link>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter();
const form = ref({ email: '', password: '' });
const error = ref('');
const loading = ref(false);

const login = async () => {
  error.value = '';
  loading.value = true;
  
  try {
    const response = await axios.post('/api/auth/login', form.value);
    localStorage.setItem('auth_token', response.data.access_token);
    localStorage.setItem('user', JSON.stringify(response.data.user));
    
    // Configurer le header pour les requêtes futures
    axios.defaults.headers.common['Authorization'] = `Bearer ${response.data.access_token}`;
    
    router.push({ name: 'Chat' });
  } catch (err) {
    if (err.response && err.response.data.errors) {
      error.value = Object.values(err.response.data.errors)[0][0];
    } else if (err.response && err.response.data.message) {
      error.value = err.response.data.message;
    } else {
      error.value = 'Erreur lors de la connexion. Veuillez réessayer.';
    }
  } finally {
    loading.value = false;
  }
};
</script>
