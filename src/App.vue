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
        title: 'valley on mountain',
        platform: 'ios'
      }
    }
  }
})
const { mutate: updateGame } = useMutation(updateMutation, () => {
  return {
    variables: {
      id:2398,
      edits: {
        title: 'valley',
        platform: 'ios'
      }
    }
  }
})

const { mutate: deleteGame } = useMutation(deleteMutation, () => {
  return {
    variables: {
      id: 3526
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

</script>

<template>
  <h1>Games</h1>
  <button @click="handleDelete()">Delete</button>
  <button @click="handleAdd()">Add</button>
  <button @click="handleUpdate()">Update</button>
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
