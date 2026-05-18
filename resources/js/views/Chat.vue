<template>
  <div class="flex h-screen w-full bg-[#0f172a] text-slate-200 overflow-hidden font-sans selection:bg-indigo-500/30">
    <!-- Sidebar / Liste des conversations -->
    <div class="w-80 lg:w-96 glass-chat border-r border-white/5 flex flex-col z-20 relative">
      <div class="p-6 border-b border-white/5 flex justify-between items-center bg-white/[0.02]">
        <div class="flex items-center gap-3">
          <div class="relative group">
            <img :src="currentUser.avatar_url || `https://ui-avatars.com/api/?name=${currentUser.name}&background=6366f1&color=fff`" class="w-11 h-11 rounded-2xl object-cover ring-2 ring-indigo-500/30 transition-all duration-300 group-hover:ring-indigo-500/60" alt="Avatar"/>
            <div class="absolute -bottom-0.5 -right-0.5 w-3.5 h-3.5 bg-green-500 border-2 border-[#0f172a] rounded-full"></div>
          </div>
          <div>
            <h2 class="text-lg font-bold text-white leading-tight">Messages</h2>
            <div class="flex items-center gap-1.5">
              <span class="w-1.5 h-1.5 bg-green-500 rounded-full animate-pulse"></span>
              <p class="text-[10px] text-slate-400 font-bold uppercase tracking-wider">Actif</p>
            </div>
          </div>
        </div>
        <button @click="logout" title="Déconnexion" class="p-2.5 hover:bg-red-500/10 text-slate-400 hover:text-red-400 rounded-xl transition-all duration-300">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1"></path></svg>
        </button>
      </div>
      
      <!-- Recherche / Nouveau Chat -->
      <div class="p-5 space-y-4">
        <div class="relative group">
          <div class="absolute inset-y-0 left-3 flex items-center pointer-events-none text-slate-500 group-focus-within:text-indigo-400 transition-colors">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg>
          </div>
          <select v-model="selectedNewUser" @change="startConversation" class="w-full pl-10 pr-4 py-3 bg-white/5 border border-white/10 rounded-2xl text-sm text-slate-200 focus:ring-2 focus:ring-indigo-500/50 focus:border-indigo-500/50 outline-none transition-all appearance-none cursor-pointer hover:bg-white/10">
            <option value="" class="bg-[#1e293b]">Démarrer une discussion...</option>
            <option v-for="user in availableUsers" :key="user.id" :value="user.id" class="bg-[#1e293b]">
              {{ user.name }}
            </option>
          </select>
        </div>
      </div>

      <!-- Conversations actives -->
      <div class="flex-1 overflow-y-auto px-3 space-y-1 pb-6">
        <button 
          v-for="conv in conversations" :key="conv.id"
          @click="selectConversation(conv)"
          :class="['w-full text-left p-4 border-b hover:bg-gray-100 transition duration-150 flex items-center gap-4', selectedConversation?.id === conv.id ? 'bg-indigo-50 border-l-4 border-indigo-500' : '']"
        >
          <div class="relative">
            <div class="w-12 h-12 rounded-full bg-indigo-200 text-indigo-700 flex items-center justify-center font-bold text-lg shadow-inner">
              {{ getConversationName(conv).charAt(0).toUpperCase() }}
            </div>
            <!-- Pastille de notification "non lu" -->
            <div v-if="conv.unread_count > 0" class="absolute -top-1 -right-1 bg-red-500 text-white text-xs font-bold w-5 h-5 rounded-full flex items-center justify-center border-2 border-white">
              {{ conv.unread_count }}
            </div>
          </div>
          <div class="flex-1 min-w-0">
            <h3 class="font-semibold text-gray-800 truncate text-base">{{ getConversationName(conv) }}</h3>
            <p class="text-sm text-gray-500 truncate" v-if="conv.last_message && conv.last_message.length">
              {{ conv.last_message[0].body || '📎 Pièce jointe' }}
            </p>
          </div>
        </button>
      </div>
    </div>

    <!-- Zone de messages -->
    <div class="flex-1 flex flex-col bg-slate-50 relative" v-if="selectedConversation">
      <!-- Header de la conversation -->
      <div class="p-4 bg-white border-b shadow-sm z-10 flex items-center gap-3">
        <div class="w-10 h-10 rounded-full bg-indigo-200 text-indigo-700 flex items-center justify-center font-bold">
          {{ getConversationName(selectedConversation).charAt(0).toUpperCase() }}
        </div>
        <h2 class="text-xl font-bold flex items-center gap-2 text-gray-800">
          {{ getConversationName(selectedConversation) }}
        </h2>
      </div>

      <!-- Messages -->
      <div class="flex-1 overflow-y-auto p-6 space-y-6" id="messages-container">
        <div 
          v-for="msg in messages" :key="msg.id"
          :class="['flex w-full', msg.user_id === currentUser.id ? 'justify-end' : 'justify-start']"
        >
          <div 
            :class="['max-w-[70%] rounded-2xl p-4 shadow-md text-[15px] leading-relaxed relative', msg.user_id === currentUser.id ? 'bg-indigo-600 text-white rounded-tr-none' : 'bg-white text-gray-800 border border-gray-100 rounded-tl-none']"
          >
            <!-- Nom de l'expéditeur si c'est un groupe -->
            <div v-if="selectedConversation.type === 'group' && msg.user_id !== currentUser.id" class="text-xs font-bold text-indigo-500 mb-1">
              {{ msg.sender?.name }}
            </div>
            
            <p v-if="msg.body">{{ msg.body }}</p>

            <!-- Pièce jointe -->
            <div v-if="msg.file_path" class="mt-2 text-sm">
              <a :href="'/storage/' + msg.file_path" target="_blank" :class="['flex items-center gap-2 underline underline-offset-2', msg.user_id === currentUser.id ? 'text-indigo-200 hover:text-white' : 'text-indigo-600 hover:text-indigo-800']">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.172 7l-6.586 6.586a2 2 0 102.828 2.828l6.414-6.586a4 4 0 00-5.656-5.656l-6.415 6.585a6 6 0 108.486 8.486L20.5 13"></path></svg>
                Voir le fichier joint
              </a>
            </div>

            <!-- Date/Heure -->
            <div :class="['text-[10px] mt-2 text-right opacity-70', msg.user_id === currentUser.id ? 'text-indigo-100' : 'text-gray-500']">
              {{ new Date(msg.created_at).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }) }}
            </div>
          </div>
        </div>
      </div>

      <!-- Champ de saisie -->
      <div class="p-4 bg-white border-t rounded-tl-3xl shadow-[0_-4px_6px_-1px_rgba(0,0,0,0.02)]">
        <form @submit.prevent="sendMessage" class="flex gap-3 items-center max-w-4xl mx-auto">
          <input 
            v-model="newMessage"
            type="text" 
            placeholder="Écrivez un message..." 
            class="flex-1 p-3.5 bg-gray-50 border border-transparent rounded-full focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:bg-white transition shadow-inner"
          />
          <button 
            type="submit" 
            :disabled="!newMessage.trim() || sending"
            class="bg-indigo-600 hover:bg-indigo-700 disabled:bg-gray-400 text-white rounded-full p-3 w-12 h-12 flex items-center justify-center shadow-md transition transform hover:scale-105 active:scale-95 flex-shrink-0"
          >
            <svg class="w-5 h-5 rotate-90 ml-1" fill="currentColor" viewBox="0 0 20 20"><path d="M10.894 2.553a1 1 0 00-1.788 0l-7 14a1 1 0 001.169 1.409l5-1.429A1 1 0 009 15.571V11a1 1 0 112 0v4.571a1 1 0 00.725.962l5 1.428a1 1 0 001.17-1.408l-7-14z"></path></svg>
          </button>
      <!-- Input Bar -->
      <div class="px-8 pb-8 pt-4">
        <form @submit.prevent="sendMessage" class="glass-chat rounded-[2.5rem] p-2 flex items-center gap-2 border border-white/5 shadow-[0_20px_50px_rgba(0,0,0,0.3)] ring-1 ring-white/5">
          <button type="button" class="w-12 h-12 flex items-center justify-center text-slate-500 hover:text-indigo-400 transition-colors rounded-full hover:bg-white/5">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path></svg>
          </button>
          <input 
            v-model="newMessage"
            type="text" 
            placeholder="Écrivez votre message ici..." 
            class="flex-1 bg-transparent border-none py-3 text-[15px] focus:ring-0 placeholder-slate-500 text-white font-medium ml-2"
          />
          <div class="flex items-center pr-1">
             <button 
              type="submit" 
              :disabled="!newMessage.trim() || sending"
              class="relative bg-gradient-to-tr from-indigo-500 to-indigo-700 hover:from-indigo-600 hover:to-purple-700 disabled:from-slate-800 disabled:to-slate-900 text-white rounded-full w-14 h-14 flex items-center justify-center shadow-xl shadow-indigo-500/20 transition-all duration-300 scale-90 hover:scale-100 active:scale-95 disabled:scale-90"
            >
              <svg :class="['w-6 h-6 transition-transform', sending ? 'animate-bounce' : 'group-hover:translate-x-1 rotate-45 -mt-1 -mr-1']" fill="currentColor" viewBox="0 0 20 20"><path d="M10.894 2.553a1 1 0 00-1.788 0l-7 14a1 1 0 001.169 1.409l5-1.429A1 1 0 009 15.571V11a1 1 0 112 0v4.571a1 1 0 00.725.962l5 1.428a1 1 0 001.17-1.408l-7-14z"></path></svg>
            </button>
          </div>
        </form>
      </div>
    </div>
    
    <!-- Empty State Moderne -->
    <div class="flex-1 flex flex-col items-center justify-center p-12 overflow-hidden relative" v-else>
      <div class="absolute -top-1/4 -right-1/4 w-[600px] h-[600px] bg-indigo-600/5 blur-[160px] rounded-full"></div>
      <div class="absolute -bottom-1/4 -left-1/4 w-[600px] h-[600px] bg-purple-600/5 blur-[160px] rounded-full"></div>
      
      <div class="relative mb-12">
        <div class="bg-gradient-to-br from-indigo-600/20 to-purple-600/20 p-12 rounded-[4rem] border border-white/5 shadow-2xl backdrop-blur-xl transform rotate-3 hover:rotate-0 transition-transform duration-700">
          <svg class="w-24 h-24 text-indigo-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"></path></svg>
        </div>
        <div class="absolute -top-6 -right-6 w-16 h-16 bg-indigo-500 rounded-full blur-3xl opacity-30 animate-pulse"></div>
      </div>
      
      <h2 class="text-5xl font-black text-white mb-6 tracking-tight text-center leading-[1.1]">
        Prêt à briser <br/> <span class="bg-gradient-to-r from-indigo-400 to-purple-400 bg-clip-text text-transparent">la glace ?</span>
      </h2>
      <p class="text-slate-400 text-lg font-semibold text-center max-w-sm leading-relaxed opacity-80">
        Choisissez un contact à gauche pour lancer une discussion sécurisée et instantanée.
      </p>
      
      <div class="mt-16 flex flex-col items-center gap-6">
        <div class="flex -space-x-4">
          <div v-for="i in 5" :key="i" class="w-12 h-12 rounded-2xl border-[3px] border-[#0f172a] bg-slate-800 flex items-center justify-center text-xs font-black text-slate-500 shadow-xl transition-transform hover:-translate-y-2 hover:z-30 cursor-pointer">
            {{ String.fromCharCode(64 + i) }}
          </div>
        </div>
        <div class="px-6 py-2 bg-white/[0.03] border border-white/5 rounded-full">
          <span class="text-[11px] text-indigo-400 font-black uppercase tracking-widest">+ Vos amis vous attendent</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, nextTick } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter();
const currentUser = ref(JSON.parse(localStorage.getItem('user') || '{}'));

const conversations = ref([]);
const availableUsers = ref([]);
const selectedNewUser = ref('');

const selectedConversation = ref(null);
const messages = ref([]);
const newMessage = ref('');
const sending = ref(false);

let pollingInterval = null;

onMounted(async () => {
  await fetchConversations();
  await fetchUsers();
  
  // HTTP POLLING: toutes les 3 secondes
  pollingInterval = setInterval(() => {
    if (selectedConversation.value) {
      pollMessages();
    }
    fetchConversations();
  }, 3000);
});

onUnmounted(() => {
  if (pollingInterval) clearInterval(pollingInterval);
});

const getConversationName = (conv) => {
  if (conv.type === 'group') return conv.name;
  const otherUser = conv.users?.find(u => u.id !== currentUser.value.id);
  return otherUser ? otherUser.name : 'Utilisateur inconnu';
};

const fetchConversations = async () => {
  try {
    const res = await axios.get('/api/conversations');
    conversations.value = res.data;
  } catch (err) {
    if (err.response?.status === 401) logout();
  }
};

const fetchUsers = async () => {
  try {
    const res = await axios.get('/api/users');
    availableUsers.value = res.data;
  } catch (err) {
    console.error(err);
  }
};

const startConversation = async () => {
  if (!selectedNewUser.value) return;
  
  try {
    const res = await axios.post('/api/conversations', {
      user_id: selectedNewUser.value,
      is_group: false
    });
    
    await fetchConversations();
    selectConversation(res.data);
    selectedNewUser.value = '';
  } catch (err) {
    console.error(err);
  }
};

const selectConversation = async (conv) => {
  selectedConversation.value = conv;
  await pollMessages(true);
};

const pollMessages = async (forceScroll = false) => {
  if (!selectedConversation.value) return;
  
  try {
    const res = await axios.get(`/api/conversations/${selectedConversation.value.id}/messages`);
    const newMessages = res.data.data.reverse();
    
    if (newMessages.length !== messages.value.length) {
      messages.value = newMessages;
      if (forceScroll || isScrolledToBottom()) {
        scrollToBottom();
      }
    }
  } catch (err) {
    console.error(err);
  }
};

const sendMessage = async () => {
  if (!newMessage.value.trim() || !selectedConversation.value) return;
  
  sending.value = true;
  const body = newMessage.value;
  
  // Optimistic UI update
  const tempMsg = {
    id: Date.now(),
    body: body,
    user_id: currentUser.value.id,
    created_at: new Date().toISOString()
  };
  messages.value.push(tempMsg);
  newMessage.value = '';
  scrollToBottom();

  try {
    await axios.post(`/api/conversations/${selectedConversation.value.id}/messages`, {
      body: body
    });
    await fetchConversations();
  } catch (err) {
    messages.value = messages.value.filter(m => m.id !== tempMsg.id); 
  } finally {
    sending.value = false;
  }
};

const isScrolledToBottom = () => {
  const container = document.getElementById('messages-container');
  if (!container) return false;
  return container.scrollHeight - container.clientHeight <= container.scrollTop + 50;
};

const scrollToBottom = () => {
  nextTick(() => {
    const container = document.getElementById('messages-container');
    if (container) {
      container.scrollTop = container.scrollHeight;
    }
  });
};

const logout = async () => {
  try {
    await axios.post('/api/auth/logout');
  } catch (e) {
  }
  localStorage.removeItem('auth_token');
  localStorage.removeItem('user');
  router.push({ name: 'Login' });
};
</script>
