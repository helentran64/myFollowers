<template>
  <v-container fill-height class="d-flex justify-center align-center">
    <div class="pa-4">
      <h1 class="text-center">View who is not following you back</h1>

      <h3>Upload your followers list</h3>
      <v-row class="mb-4">
        <v-col cols="12">
          <v-file-upload
            clearable
            density="compact"
            type="file"
            @change="onFollowersFileChange"
            accept=".json"
          />
        </v-col>
      </v-row>

      <h3>Upload your following list</h3>
      <v-row class="mb-4">
        <v-col cols="12">
          <v-file-upload
            clearable
            density="compact"
            type="file"
            @change="onFollowingFileChange"
            accept=".json"
          />
        </v-col>
      </v-row>

      <div class="d-flex flex-column align-center">
        <v-btn @click="checkUnfollowers" color="primary" :disabled="!canCheckFollowers">
          Check Unfollowers
        </v-btn>

        <div v-if="resultsChecked" class="mt-4 text-center">
          <h3>Results:</h3>
          <ul>
            <li v-for="user in unfollowers" :key="user">{{ user }}</li>
            <li v-if="unfollowers.length === 0">Everyone follows you back!</li>
          </ul>
        </div>
      </div>
    </div>
  </v-container>
</template>
<script setup lang="ts">
import { ref, computed } from 'vue'
import { VFileUpload } from 'vuetify/labs/VFileUpload'

// --- File Refs ---
const followersFile = ref<File | null>(null)
const followingFile = ref<File | null>(null)

// --- Reactive State ---
const followers = ref<string[]>([])
const following = ref<string[]>([])
const unfollowers = ref<string[]>([])
const resultsChecked = ref(false)

// --- TypeScript Interfaces ---
interface FollowerItem {
  string_list_data: {
    href: string
    value: string
    timestamp: number
  }[]
}

interface FollowingItem {
  title: string
  string_list_data: {
    href: string
    timestamp: number
  }[]
}

type FollowersJson = FollowerItem[]
interface FollowingJson {
  relationships_following: FollowingItem[]
}

// --- Helper to Read JSON from File ---
const readJsonFile = async <T,>(file: File): Promise<T> => {
  try {
    const result = await new Promise<string>((resolve, reject) => {
      const reader = new FileReader()
      reader.onload = () => {
        if (typeof reader.result === 'string') {
          resolve(reader.result)
        } else {
          reject(new Error('FileReader result is not a string'))
        }
      }
      reader.onerror = () => reject(reader.error)
      reader.readAsText(file)
    })

    try {
      return JSON.parse(result) as T
    } catch (err) {
      throw new Error('Invalid JSON: ' + (err as Error).message)
    }
  } catch (err) {
    console.error('Error reading file:', err)
    throw err
  }
}

// --- File Change Handlers ---
const onFollowersFileChange = async (event: Event) => {
  const target = event.target as HTMLInputElement
  if (!target.files?.length) return
  followersFile.value = target.files[0]!

  try {
    const data = await readJsonFile<FollowersJson>(followersFile.value)
    followers.value = data.map((item) => item.string_list_data.map((obj) => obj.value)).flat()
  } catch (err) {
    alert('Error reading followers file: ' + (err as Error).message)
  }
}

const onFollowingFileChange = async (event: Event) => {
  const target = event.target as HTMLInputElement
  if (!target.files?.length) return
  followingFile.value = target.files[0]!

  try {
    const data = await readJsonFile<FollowingJson>(followingFile.value)
    following.value = data.relationships_following.map((item) => item.title)
  } catch (err) {
    alert('Error reading following file: ' + (err as Error).message)
  }
}

// --- Compare Followers and Following ---
const checkUnfollowers = () => {
  if (!followers.value.length || !following.value.length) {
    alert('Please upload both files first.')
    return
  }
  unfollowers.value = following.value.filter((f) => !followers.value.includes(f))
  resultsChecked.value = true
}

// --- Computed Property to Enable Button ---
const canCheckFollowers = computed(() => {
  return followers.value.length > 0 && following.value.length > 0
})
</script>

<style scoped>
ul {
  list-style-type: none;
  padding: 0;
  max-height: 200px;
  overflow-y: auto;
  width: 100%;
}
</style>
