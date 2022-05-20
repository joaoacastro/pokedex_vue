<template>
  <v-app>
    <v-container class="img">
      <img
        alt="Pokedex logo"
        src="./img/logo-pokedex.svg"
        style="width: 400px"
      />
    </v-container>

    <v-container>
      <v-container>
        <v-text-field
          v-model="search"
          label="Pesquisar"
          placeholder="Nome do Pokemon"
          solo
        ></v-text-field>

        <v-row>
          <v-col
            cols="3"
            v-for="pokemon in filtered_pokemons"
            :key="pokemon.name"
          >
            <v-card @click="show_pokemon(get_id(pokemon))">
              <v-container class="card">
                <v-row class="table-card">
                  <img
                    :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${get_id(
                      pokemon
                    )}.png`"
                    :alt="pokemon.name"
                    width="40%"
                  />
                </v-row>
                <h2 class="text-center">{{ get_name(pokemon) }}</h2>
              </v-container>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-container>

    <v-dialog v-model="show_dialog" width="800">
      <v-card v-if="selected_pokemon" class="px-2">
        <v-container>
          <v-row class="table">
            <v-col cols="4">
              <img
                :src="`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${selected_pokemon.id}.png`"
                :alt="selected_pokemon.name"
                width="80%"
              />
            </v-col>
            <v-col cols="8">
              <h1>{{ get_name(selected_pokemon) }}</h1>
              <v-chip
                color="rgb(253, 254, 197)"
                class="type"
                v-for="type in selected_pokemon.types"
                :key="type.slot"
              >
                {{ type.type.name }}
              </v-chip>

              <v-divider class="divider" />

              <v-chip color="rgb(219, 254, 197)" class="height">
                Altura {{ selected_pokemon.height * 2.54 }} cm
              </v-chip>

              <v-chip color="rgb(219, 254, 197)" class="weight">
                Peso {{ (selected_pokemon.weight * 0.453592).toFixed(2) }} kg
              </v-chip>
            </v-col>
          </v-row>

          <v-chip color="#DA4944" class="hp">
            Hp {{ selected_pokemon.stats[0].base_stat }}
          </v-chip>

          <v-chip color="#FF9E00" class="attack">
            Attack {{ selected_pokemon.stats[1].base_stat }}
          </v-chip>

          <v-chip color="#CC7F02" class="defense">
            Special Attack {{ selected_pokemon.stats[3].base_stat }}
          </v-chip>

          <v-chip color="#008BFF" class="defense">
            Defense {{ selected_pokemon.stats[2].base_stat }}
          </v-chip>

          <v-chip color="#026EC8" class="defense">
            Special Defense {{ selected_pokemon.stats[4].base_stat }}
          </v-chip>

          <h2>Moves</h2>

          <v-simple-table>
            <template>
              <thead>
                <tr>
                  <th class="text-left">Name</th>
                  <th class="text-left">Level</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="item in filter_moves(selected_pokemon)"
                  :key="item.move.name"
                >
                  <td>
                    {{
                      item.move.name.charAt(0).toUpperCase() +
                      item.move.name.slice(1)
                    }}
                  </td>
                  <td>{{ get_move_level(item) }}</td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-container>
      </v-card>
    </v-dialog>
    <footer>.:Thanks for using my website | Developed by João A. Castro:.</footer>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  name: "App",

  components: {},

  data() {
    return {
      pokemons: [],
      search: "",
      show_dialog: false,
      selected_pokemon: null,
    };
  },

  mounted() {
    axios
      .get("https://pokeapi.co/api/v2/pokemon?limit=151")
      .then((response) => {
        this.pokemons = response.data.results;
      });
  },
  methods: {
    get_id(pokemon) {
      return Number(pokemon.url.split("/")[6]);
    },
    get_name(pokemon) {
      return pokemon.name.charAt(0).toUpperCase() + pokemon.name.slice(1);
    },
    show_pokemon(id) {
      axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`).then((response) => {
        this.selected_pokemon = response.data;
        this.show_dialog = !this.show_dialog;
      });
    },
    get_move_level(move) {
      for (let version of move.version_group_details) {
        if (
          version.version_group.name == "sword-shield" &&
          version.move_learn_method.name == "level-up"
        ) {
          return version.level_learned_at;
        }
      }
      return 0;
    },
    filter_moves(pokemon) {
      return pokemon.moves.filter((item) => {
        let include = false;
        for (let version of item.version_group_details) {
          if (
            version.version_group.name == "sword-shield" &&
            version.move_learn_method.name == "level-up"
          ) {
            include = true;
          }
        }
        return include;
      });
    },
  },
  computed: {
    filtered_pokemons() {
      return this.pokemons.filter((item) => {
        return item.name.includes(this.search);
      });
    },
  },
};
</script>

<style>
#app {
  background-color: #d3bffa;
}
.attack {
  margin-left: 10px;
}
.card {
  background-color: #bcaae0;
  border: solid 1px black;
}
.card:hover {
  background-color: #ad8bf1;
}
.defense {
  margin-left: 10px;
}
.divider {
  margin: 20px;
}
footer {
  display: flex;
  justify-content: center;
  color: rgb(255, 255, 255);
  font-family: "Courier New", Courier, monospace;
  font-style: italic;
}
.hp {
  margin-left: 10px;
}
.height {
  margin-left: 10px;
}
.img {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}
.table-card {
  display: flex;
  justify-content: center;
  cursor: pointer;
}
.table {
  display: flex;
  align-content: center;
}
.type {
  margin-top: 10px;
  margin-left: 10px;
}
.weight {
  margin-left: 10px;
}
</style>
