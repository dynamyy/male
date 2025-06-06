<template>
  <v-container>
    <v-row>
      <v-col>
        <h1>Klubid</h1>
      </v-col>
    </v-row>
    <v-row class="mb-4">
      <v-col
        cols="12"
        sm="3"
      >
        <v-text-field
          v-model="searchName"
          label="Otsi nime järgi"
          clearable
          @click:clear="clearSearch"
        />
      </v-col>

      <v-col
        cols="12"
        sm="1"
      >
        <v-select
          v-model="resultsPerPage"
          :items="[10, 20, 50]"
          label="Tulemusi lehel"
          :menu-props="{ maxHeight: 200 }"
        />
      </v-col>

      <v-col
        cols="12"
        sm="2"
      >
        <v-select
          v-model="sortBy"
          :items="['Reiting', 'Nimi', 'Liikmete arv']"
          label="Järjesta"
        />
      </v-col>

      <v-col
        cols="12"
        sm="6"
        class="d-flex justify-end"
      >
        <v-btn
          color="primary"
          @click="openAddClubDialog"
        >
          Lisa uus klubi
        </v-btn>
      </v-col>
    </v-row>

    <v-row>
      <v-col
        v-for="club in filteredClubs"
        :key="club.id"
        cols="12"
        sm="6"
        md="4"
      >
        <v-card
          outlined
          hover
          class="mb-4"
          @click="goToClubDetails(club.id)"
        >
          <v-card-title>{{ club.name }}</v-card-title>
          <v-card-subtitle>
            <strong>Keskmine reiting:</strong> {{ club.averageRating }}
            | <strong>Liikmeid:</strong> {{ club.membersCount }}
          </v-card-subtitle>
          <v-card-text />
        </v-card>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <v-pagination
          v-model="page"
          :length="pageCount"
        />
      </v-col>
    </v-row>
    <AddClubDialog
      v-model:show-dialog="showAddClubDialog"
      @update:show-dialog="updateShowClubDialog"
      @club-updated="fetchAllClubsData"
    />
  </v-container>
</template>

<script>
import {fetchAllClubs} from "@/wrapper/clubsApiWrapper.js";
import AddClubDialog from "@/components/clubs/AddClubDialog.vue";

export default {
  name: "ClubsPage",
  components: {AddClubDialog},
  data() {
    return {
      searchName: "",
      resultsPerPage: 10,
      sortBy: "Reiting",
      page: 1,
      clubs: [],
      showAddClubDialog: false,
    }
  },
  computed: {
    filteredClubs() {
      let filtered = this.clubs.filter((club) =>
        club.name.toLowerCase().includes(this.searchName.toLowerCase())
      )

      switch (this.sortBy) {
        case "Nimi":
          filtered.sort((a, b) => a.name.localeCompare(b.name))
          break
        case "Reiting":
          filtered.sort((a, b) => b.averageRating - a.averageRating)
          break
        case "Liikmete arv":
          filtered.sort((a, b) => b.membersCount - a.membersCount)
          break
      }

      const startIndex = (this.page - 1) * this.resultsPerPage
      const endIndex = startIndex + this.resultsPerPage

      return filtered.slice(startIndex, endIndex)
    },
    totalFiltered() {
      return this.clubs.filter((club) =>
        club.name.toLowerCase().includes(this.searchName.toLowerCase())
      ).length
    },
    pageCount() {
      return Math.ceil(this.totalFiltered / this.resultsPerPage)
    },
  },
  watch: {
    sortBy() {
      this.page = 1
    },
    resultsPerPage() {
      this.page = 1
    },
  },
  created() {
    this.$watch(
      () => this.$route.params.id,
      this.fetchAllClubsData,
      {immediate: true}
    )
  },
  methods: {
    async fetchAllClubsData() {
      this.clubs = await fetchAllClubs()
    },

    goToClubDetails(clubId) {
      this.$router.push(`/klubid/${clubId}`)
    },

    openAddClubDialog() {
      this.showAddClubDialog = true;
    },

    updateShowClubDialog(value) {
      this.showAddClubDialog = value
    },

    clearSearch() {
      this.searchName = "";
    },
  },
}
</script>

<style scoped>
</style>
