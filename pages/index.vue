<script setup>
const user = useSupabaseUser()
</script>

<template>
  <section class="text-center py-20 bg-gray-50">
    <h2 class="text-6xl font-bold">Welcome to <span class="text-blue-600">Quizlet</span></h2>
    <p class="text-gray-600 mt-4">Test your knowledge with interactive quizzes. Create or take quizzes now!</p>
    <div class="mt-6">
      <button class="bg-blue-600 text-white px-6 py-2 rounded-lg mr-4">
        <NuxtLink href="/quizzes">Browse Quizzes</NuxtLink>
      </button>
      <button v-if="!user" class="border border-blue-600 text-blue-600 px-6 py-2 rounded-lg">
        <NuxtLink href="/sign-up">Sign Up Free</NuxtLink>
      </button>
    </div>
  </section>

  <section class="py-16">
    <div class="container mx-auto">
      <h2 class="text-3xl font-bold text-center">Everything you need for interactive learning</h2>
      <p class="text-gray-600 mt-4 text-center">Our platform provides all the tools you need to create, share, and take quizzes on any topic.</p>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mt-8">
        <div class="p-5 border border-gray-200 rounded-lg shadow-md space-y-2">
          <div class="w-12 h-12 p-2 bg-blue-100 rounded flex items-center justify-center">
            <Icon name="material-symbols:smart-toy-outline" size="24px" class="text-blue-600" />
          </div>
          <h3 class="font-semibold">Create Custom Quizzes</h3>
          <p class="text-gray-600">Easily create custom quizzes with multiple-choice questions.</p>
        </div>
        <div class="p-5 border border-gray-200 rounded-lg shadow-md space-y-2">
          <div class="w-12 h-12 p-2 bg-blue-100 rounded flex items-center justify-center">
            <Icon name="material-symbols:award-star-outline" size="24px" class="text-blue-600" />
          </div>
          <h3 class="font-semibold">Track Your Progress</h3>
          <p class="text-gray-600">Keep track of your quiz results and progress over time.</p>
        </div>
        <div class="p-5 border border-gray-200 rounded-lg shadow-md space-y-2">
          <div class="w-12 h-12 p-2 bg-blue-100 rounded flex items-center justify-center">
            <Icon name="line-md:account" size="24px" class="text-blue-600" />
          </div>
          <h3 class="font-semibold">Share With Others</h3>
          <p class="text-gray-600">Share your quizzes with friends or the community.</p>
        </div>
      </div>
    </div>
  </section>

  <section class="py-16 bg-gray-50 text-center">
    <h2 class="text-3xl font-bold">How It Works</h2>
    <p class="text-gray-600 mt-4">Get started with QuizApp in just a few simple steps</p>
    <div class="flex justify-center mt-8 space-x-8">
      <div class="p-6 space-y-3 w-100">
        <div class="bg-blue-600 p-4 w-12 h-12 rounded-full flex items-center justify-center mx-auto">
          <p class="text-white text-2xl">1</p>
        </div>
        <p>Sign Up for a free account.</p>
        <p class="text-gray-500">Create a free account to access all features of QuizApp.</p>
      </div>
      <div class="p-6 space-y-3 w-100">
        <div class="bg-blue-600 p-4 w-12 h-12 rounded-full flex items-center justify-center mx-auto">
          <p class="text-white text-2xl">2</p>
        </div>
        <p>Create or Browse quizzes.</p>
        <p class="text-gray-500">Create your own quizzes or browse existing ones from our collection.</p>
      </div>
      <div class="p-6 space-y-3 w-100">
        <div class="bg-blue-600 p-4 w-12 h-12 rounded-full flex items-center justify-center mx-auto">
          <p class="text-white text-2xl">3</p>
        </div>
        <p>Take quizzes and track progress.</p>
        <p class="text-gray-500">Test your knowledge, get instant results, and track your progress.</p>
      </div>
    </div>
  </section>

  <section class="py-16 text-center space-y-20">
    <h2 class="text-3xl font-bold">Why Users Love Quizlet</h2>
    <div class="flex justify-center space-x-50">
      <div>
        <h3 class="text-4xl font-bold text-blue-600">1000+</h3>
        <p class="text-gray-500">Quizzes Created</p>
      </div>
      <div>
        <h3 class="text-4xl font-bold text-blue-600">5000+</h3>
        <p class="text-gray-500">Active Users</p>
      </div>
      <div>
        <h3 class="text-4xl font-bold text-blue-600">50+</h3>
        <p class="text-gray-500">Categories</p>
      </div>
    </div>
    <div class="max-w-5xl mx-auto mt-8 bg-gray-100 p-6 rounded-lg space-y-4">
      <Icon v-for="i in 5" name="material-symbols:star-rate" size="24px" class="text-yellow-400" />
      <p class="italic text-gray-600">"QuizApp has transformed how I study for exams. Creating quizzes helps me retain information better, and taking quizzes from others exposes me to new perspectives."</p>
      <p class="font-bold">Sarah Johnson, University Student</p>
    </div>
  </section>

  <section class="bg-blue-600 flex justify-around text-white text-center py-12">
    <div class="space-y-2 flex flex-col items-start">
      <h2 class="text-4xl font-bold">Ready to get started?</h2>
      <h2 class="text-4xl font-bold">Join Quizlet today.</h2>
    </div>
    <div class="mt-6">
      <button class="bg-white text-blue-600 px-6 py-2 rounded-lg mr-4">
        <NuxtLink href="/sign-up">Get Started</NuxtLink>
      </button>
      <button class="border border-white px-6 py-2 rounded-lg">
        <NuxtLink href="/quizzes">
          Browse Quizzes
        </NuxtLink>
      </button>
    </div>
  </section>
</template>
