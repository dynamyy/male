<template>
  <v-row>
    <v-col>
      <v-btn
        color="primary"
        @click="openAddMatchDialog"
      >
        Lisa partii
      </v-btn>
    </v-col>
  </v-row>
  <v-row>
    <v-col>
      <v-data-table
        :headers="headers"
        :items="matches"
        :search="searchName"
        no-data-text="Partiisid ei leitud"
        item-key="id"
        class="elevation-1"
      >
        <template #top>
          <v-toolbar
            flat
            dense
            color="secondary"
          >
            <v-toolbar-title>Partiid</v-toolbar-title>
            <v-spacer />
            <v-text-field
              v-model="searchName"
              label="Otsi mängija nime järgi"
              clearable
              dense
              outlined
              hide-details
              style="max-width: 300px;"
            />
          </v-toolbar>
        </template>

        <template #item.action="{ item }">
          <v-row
            align="center"
            justify="end"
          >
            <v-btn
              color="secondary"
              @click.stop="openUpdateMatchDialog(item)"
            >
              Muuda
            </v-btn>
          </v-row>
        </template>
      </v-data-table>
    </v-col>
  </v-row>

  <AddMatchDialog
    v-model:show-dialog="showAddDialog"
    :tournament-id="tournamentId"
    @match-updated="fetchAllMatchesData"
    @update:show-dialog="handleShowAddDialog"
  />

  <AddMatchDialog
    v-if="selectedMatch"
    v-model:show-dialog="showUpdateDialog"
    :tournament-id="tournamentId"
    :is-update="true"
    :match-id="selectedMatch.id"
    @match-updated="fetchAllMatchesData"
    @update:show-dialog="handleShowUpdateDialog"
  />
</template>

<script>
import {fetchMatchesByTournamentId} from "@/wrapper/matchesApiWrapper.js";
import AddMatchDialog from "@/components/tournaments/AddMatchDialog.vue";

export default {
  components: {AddMatchDialog},

  props: {
    tournamentId: {
      type: String,
      required: true,
    }
  },
  data() {
    return {
      headers: [
        {title: 'Valge', value: 'white.fullName', sortable: true},
        {title: 'Must', value: 'black.fullName', sortable: true},
        {title: 'Algus', value: 'startTime', sortable: true},
        {title: 'Lõpp', value: 'endTime', sortable: true},
        {title: 'Võitja', value: 'winner', sortable: true},
        {value: 'action', sortable: false},
      ],
      matches: [],
      searchName: '',
      selectedMatch: null,
      showUpdateDialog: false,
      showAddDialog: false,
    }
  },

  created() {
    this.$watch(
      () => this.$route.params.id,
      this.fetchAllMatchesData,
      {immediate: true}
    )
  },

  methods: {
    async fetchAllMatchesData() {
      this.matches = await fetchMatchesByTournamentId(this.tournamentId)
    },

    openAddMatchDialog() {
      this.showAddDialog = true
    },

    openUpdateMatchDialog(match) {
      this.selectedMatch = match
      this.showUpdateDialog = true
    },

    handleShowUpdateDialog(value) {
      if (value === false) {
        this.selectedMatch = null;
      }
      this.showUpdateDialog = value;
    },

    handleShowAddDialog(value) {
      this.showAddDialog = value
    },

    nukeMatch() {
      // are you sure
    }
  }
}
</script>

<style scoped>

</style>
