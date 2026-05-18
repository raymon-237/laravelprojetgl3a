<template>
  <div class="m-auto w-full max-w-[440px]">
    <div class="glass p-10 rounded-[2.5rem] shadow-2xl relative overflow-hidden group">
      <!-- Glow effect -->
      <div class="absolute -bottom-24 -right-24 w-48 h-48 bg-purple-500/10 blur-[60px] rounded-full group-hover:bg-purple-500/20 transition-colors duration-700"></div>
      
      <div class="relative z-10">
        <div class="w-16 h-16 bg-gradient-to-tr from-purple-500 to-indigo-600 rounded-2xl flex items-center justify-center mb-8 mx-auto shadow-lg shadow-purple-500/20 -rotate-3 transition-transform hover:rotate-0 duration-500">
          <svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z"></path></svg>
        </div>
        
        <h2 class="text-3xl font-black text-center text-white mb-2 tracking-tight">Rejoignez-nous</h2>
        <p class="text-slate-400 text-center text-sm font-medium mb-10">Créez votre profil en quelques secondes</p>
        
        <form @submit.prevent="register" class="space-y-5">
          <div class="space-y-2">
            <label class="block text-[12px] font-bold text-slate-500 uppercase tracking-widest px-1">Nom complet</label>
            <input 
              v-model="form.name"
              type="text" 
              required 
              placeholder="Jean Dupont"
              class="w-full px-5 py-3.5 bg-white/5 border border-white/10 rounded-2xl text-white placeholder-slate-500 outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all font-medium"
            />
          </div>

          <div class="space-y-2">
            <label class="block text-[12px] font-bold text-slate-500 uppercase tracking-widest px-1">Adresse Email</label>
            <input 
              v-model="form.email"
              type="email" 
              required 
              placeholder="votre@email.com"
              class="w-full px-5 py-3.5 bg-white/5 border border-white/10 rounded-2xl text-white placeholder-slate-500 outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all font-medium"
            />
          </div>
          
          <div class="grid grid-cols-2 gap-4">
            <div class="space-y-2">
              <label class="block text-[12px] font-bold text-slate-500 uppercase tracking-widest px-1">Mot de passe</label>
              <input 
                v-model="form.password"
                type="password" 
                required 
                placeholder="••••••••"
                class="w-full px-5 py-3.5 bg-white/5 border border-white/10 rounded-2xl text-white placeholder-slate-500 outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all font-medium"
              />
            </div>
            <div class="space-y-2">
              <label class="block text-[12px] font-bold text-slate-500 uppercase tracking-widest px-1">Confirmation</label>
              <input 
                v-model="form.password_confirmation"
                type="password" 
                required 
                placeholder="••••••••"
                class="w-full px-5 py-3.5 bg-white/5 border border-white/10 rounded-2xl text-white placeholder-slate-500 outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all font-medium"
              />
            </div>
          </div>
          
          <div v-if="error" class="p-4 bg-red-500/10 border border-red-500/20 rounded-2xl text-red-400 text-xs font-bold text-center">
            {{ error }}
          </div>

          <button 
            type="submit" 
            :disabled="loading"
            class="w-full bg-gradient-to-tr from-purple-500 to-indigo-600 hover:from-purple-600 hover:to-indigo-700 text-white font-bold py-4 rounded-2xl transition-all shadow-xl shadow-purple-600/20 active:scale-[0.98] disabled:opacity-50 disabled:cursor-not-allowed group mt-2"
          >
            <div class="flex items-center justify-center gap-2">
              <span v-if="loading" class="w-5 h-5 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
              <span>{{ loading ? 'Création du compte...' : "S'inscrire" }}</span>
              <svg v-if="!loading" class="w-4 h-4 transform group-hover:translate-x-1 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7l5 5m0 0l-5 5m5-5H6"></path></svg>
            </div>
          </button>
        </form>
        
        <div class="mt-8 text-center text-[14px] text-slate-400 font-medium">
          Déjà membre ? 
          <router-link to="/login" class="text-purple-400 font-bold hover:text-purple-300 transition-colors px-1">Me connecter</router-link>
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
const form = ref({ name: '', email: '', password: '', password_confirmation: '' });
const error = ref('');
const loading = ref(false);

const register = async () => {
  error.value = '';
  loading.value = true;
  
  try {
    const response = await axios.post('/api/auth/register', form.value);
    localStorage.setItem('auth_token', response.data.access_token);
    localStorage.setItem('user', JSON.stringify(response.data.user));
    
    // Configurer le header pour les requêtes futures
    axios.defaults.headers.common['Authorization'] = `Bearer ${response.data.access_token}`;
    
    router.push({ name: 'Chat' });
  } catch (err) {
    if (err.response && err.response.data.errors) {
      error.value = Object.values(err.response.data.errors)[0][0];
    } else {
      error.value = 'Erreur lors de l\'inscription. Veuillez réessayer.';
    }
  } finally {
    loading.value = false;
  }
};
</script>
