<script setup lang="ts">
import {useMutation, useQuery} from '@vue/apollo-composable'
import gql from 'graphql-tag'
import { computed, ref, watchEffect } from 'vue'
import { GraphQLClient } from 'graphql-request'
import { title } from 'process'

// Queries
const allGames_QUERY = gql`
  query {
    games{
      id
      title
      platform
      reviews{
        content
        rating
      }
    },
    reviews{
      content
      rating
    },
    authors{
      name
      verified
    }
  }
` 

const endpoint = 'http://localhost:4000/'
const graphQLClient = new GraphQLClient(endpoint, {
  headers: {},
})

// Mutations
const addMutation = gql`
  mutation ($game: AddGameInput!) {
      addGame(game: $game) {
        id
        title
        platform
      }
  } 
`
const updateMutation = gql`
  mutation ($id: ID!, $edits: EditGameInput) {
      updateGame(id: $id, edits: $edits) {
        id
        title
        platform
      }
  } 
`

const deleteMutation= gql `
  mutation ($id: ID!){
    deleteGame(id: $id) {
      id
    }
  }
`

// fetch datas
const { result } = useQuery(allGames_QUERY)
const allGames = computed(() => result.value?.games?? [])
const allreviews = computed(() => result.value?.reviews?? [])
const allauthors = computed(() => result.value?.authors ?? [])
watchEffect(() => {console.log('fetched',allGames)})

const { mutate: addGame } = useMutation(addMutation, () => {
  return {
    variables: {
      game: {
        title: addTitle.value,
        platform: addPlatform.value
      }
    }
  }
})
const { mutate: updateGame } = useMutation(updateMutation, () => {
  return {
    variables: {
      id: updateId.value,
      edits: {
        title: updateTitle.value,
        platform: updatePlatform.value
      }
    }
  }
})

const { mutate: deleteGame } = useMutation(deleteMutation, () => {
  return {
    variables: {
      id: deleteId.value
    }
  }
})

async function handleAdd() {
  await addGame()
  location.reload()
}

async function handleUpdate() {
  await updateGame()
  location.reload()
}

async function handleDelete() {
  await deleteGame()
  location.reload()
}
const deleteId = ref('')
const addTitle = ref('')
const addPlatform = ref('')
const updateId = ref('')
const updateTitle = ref('')
const updatePlatform = ref('')
</script>

<template>
  <h1>Games</h1>
  <div>
    <input v-model="addTitle" placeholder="title" />
    <input v-model="addPlatform" placeholder="platform" />
    <button @click="handleAdd()">Add</button>
  </div>
 
  <div>
    <input v-model="updateId" placeholder="id" />
    <input v-model="updateTitle" placeholder="title" />
    <input v-model="updatePlatform" placeholder="platform" />
    <button @click="handleUpdate()">Update</button>
  </div>
  
  <div>
    <input v-model="deleteId" placeholder="id" />
    <button @click="handleDelete()">Delete</button>
  </div>

  #games
  <div v-for="game in allGames" :key="game.id">
    <p class="id">Id: {{ game.id }}</p>
    <p class="title">Titles: {{ game.title }}</p> 
    <p class="platform">Platforms: {{ game.platform }}</p>
    <p class="reviews">Reviews: {{ game.reviews }}</p>
  </div>

  #reviews
  <div v-for="review in allreviews" :key="review.id">
    <p class="reviews">Reviews: {{ review.content }}</p>
    <p class="reviews">Rating: {{ review.rating }}</p>
  </div>

  #authors
  <div v-for="author in allauthors" :key="author.id">
  <p class="author">Authors: {{ author.name }}</p>
  <p class="author">Verified: {{ author.verified }}</p>
  </div>


</template>

<style scoped>
p.title{
  font: 1em sans-serif;
  color: blue;
}
p.platform{
  color: green;
}
p.reviews{
  color: red;
}
p.author{
  color: blue;
}
</style>
