<script setup lang="ts">
import {useMutation, useQuery} from '@vue/apollo-composable'
import gql from 'graphql-tag'
import { computed, ref, watchEffect } from 'vue'
import { GraphQLClient } from 'graphql-request'

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

async function addGame(title: any, platform: any) {
  const mutation = gql `
    mutation addGame($games: AddGameInput!) {
      AddGameInput(games: $games) {
          title
          platform

      }
    }
    `
  const variables = {
    "title": title,
    "platform": platform
  }

  return await graphQLClient.rawRequest(mutation, variables);
}

async function game_Mutation(gameId: any, title: any, platform: any) {
  const mutation = gql`
    mutation deleteGame($id: ID!) {
      deleteGame(id: $id) {
        id
      }
    },
    mutation addGame($games: AddGameInput!) {
      AddGameInput(games: $games) {
          title
          platform
      }
    },
    mutation updateGame($id: ID!, $game: UpdateGameInput!) {
      updateGame(id: $id, game: $game) {
        id
        title
        platform
      }
    }
    `
   const variables = {
    "id": gameId,
    "title": title,
    "platform": platform
   }
  return await graphQLClient.rawRequest(mutation, variables);
}

const { result } = useQuery(allGames_QUERY)
const allGames = computed(() => result.value?.games?? [])
const allreviews = computed(() => result.value?.reviews?? [])
const allauthors = computed(() => result.value?.authors ?? [])

watchEffect(() => {
  console.log('fetched',allGames)
}
)

</script>

<template>
  <h1>Games</h1>
  <button @click="deleteGame(4)">Delete</button>
  <button @click="addGame('PES', ['PC', 'PS4','Mobile'])">Add</button>
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
