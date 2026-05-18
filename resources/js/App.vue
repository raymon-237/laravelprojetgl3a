<template>
  <div class="min-h-screen bg-[#0f172a] text-slate-200 selection:bg-indigo-500/30">
    <!-- Overlay background gradients -->
    <div class="fixed inset-0 overflow-hidden pointer-events-none">
      <div class="absolute -top-[10%] -left-[10%] w-[40%] h-[40%] rounded-full bg-indigo-500/10 blur-[120px]"></div>
      <div class="absolute -bottom-[10%] -right-[10%] w-[40%] h-[40%] rounded-full bg-purple-500/10 blur-[120px]"></div>
    </div>

    <main class="relative z-10 h-screen flex overflow-hidden">
      <router-view v-slot="{ Component }">
        <transition 
          name="fade" 
          mode="out-in"
        >
          <component :is="Component" />
        </transition>
      </router-view>
    </main>
  </div>
</template>

<script setup>
// Racine de l'application
</script>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(10px);
}
</style>
