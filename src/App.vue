<template>
    <div class="ma-5">
        <v-row>
          <v-col cols="3" class="d-flex flex-column">
            <v-form ref="form" @submit.prevent="addProduct">
              <v-text-field
                v-model="data.brand"
                label="Brand"
                variant="outlined"
                density="compact"
                :rules="[v => !!v || 'Brand is required']"
              />

              <v-text-field
                v-model="data.type"
                label="Type"
                variant="outlined"
                density="compact"
                :rules="[v => !!v || 'Type is required']"
              />

              <v-text-field
                v-model="data.model"
                label="Model"
                variant="outlined"
                density="compact"
                :rules="[v => !!v || 'Model is required']"
              />

              <v-text-field
                v-model="data.color"
                label="Color"
                variant="outlined"
                density="compact"
                :rules="[v => !!v || 'Color is required']"
              />

              <v-btn 
                color="primary"
                block
                type="submit"
              >
                ADD
              </v-btn>
            </v-form>
          </v-col>

          <v-col>
            <v-table>
              <thead>
                <tr>
                  <th>Product Id</th>
                  <th>Brand</th>
                  <th>Type</th>
                  <th>Model</th>
                  <th>Color</th>
                  <th>Action</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item,i) in tableData" :key="i">
                    <td>{{ i + 1 }}</td>
                    <td>{{ item.brand }}</td>
                    <td>{{ item.type }}</td>
                    <td>{{ item.model }}</td>
                    <td>{{ item.color }}</td>
                    <td>
                      <v-btn
                        color="warning"
                        @click="openDialog(item)"
                      >
                        update
                      </v-btn>
                      <v-btn
                        color="error"
                        @click="deleteData(item.id)"
                      >
                        delete
                      </v-btn>
                    </td>
                </tr>
              </tbody>
            </v-table>
          </v-col>
        </v-row>

        <!-- dialog -->
        <v-dialog
          v-model="dialog"
          width="400px"
        >
          <v-card>
            <v-card-title>
              UPDATE
            </v-card-title>
            <v-card-text>
              <v-form ref="form" @submit.prevent="updateProduct">
                <v-text-field
                  v-model="dialogData.brand"
                  label="Brand"
                  variant="outlined"
                  density="compact"
                  :rules="[v => !!v || 'Brand is required']"
                />
  
                <v-text-field
                  v-model="dialogData.type"
                  label="Type"
                  variant="outlined"
                  density="compact"
                  :rules="[v => !!v || 'Type is required']"
                />
  
                <v-text-field
                  v-model="dialogData.model"
                  label="Model"
                  variant="outlined"
                  density="compact"
                  :rules="[v => !!v || 'Model is required']"
                />
  
                <v-text-field
                  v-model="dialogData.color"
                  label="Color"
                  variant="outlined"
                  density="compact"
                  :rules="[v => !!v || 'Color is required']"
                />
  
                <v-btn 
                  color="primary"
                  block
                  type="submit"
                >
                  ADD
                </v-btn>
              </v-form>
            </v-card-text>
          </v-card>
        </v-dialog>
    </div>
</template>

<script setup>
  import { ref, onMounted } from 'vue'
  import axios from 'axios'

  const data = ref({})
  const form = ref(null)
  const tableData = ref([])
  const dialog = ref(false)
  const dialogData = ref({})

  const addProduct = async () =>{
    const {valid} = await form.value?.validate()
    if(valid){
      axios.post(`https://zzez7yzcy0.execute-api.ap-southeast-2.amazonaws.com/items`, data.value,{
        headers: {
          "Content-Type": "application/json",
          "x-api-key": "SECRET_KEY" // Replace with your actual secret
        }
      }).then(r =>{
        let item = r.data.data
        tableData.value.push(item)
      })
    

      await form.value.reset()
    }

  }

  const getProduct = async () => {
    const apiUrl = "https://zzez7yzcy0.execute-api.ap-southeast-2.amazonaws.com/items";

    fetch(apiUrl, {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
        "x-api-key": "SECRET_KEY" // Replace with your actual secret
      }
    })
      .then(response => response.json())
      .then(data => {
        data.sort((a, b) => new Date(a.created_at) - new Date(b.created_at));
        tableData.value = data;
      })
      .catch(error => {
        console.error("Error fetching data:", error);
      });
  }

  const deleteData = (id) => {
    axios.delete(`https://zzez7yzcy0.execute-api.ap-southeast-2.amazonaws.com/items/${id}`,{
      headers: {
          "Content-Type": "application/json",
          "x-api-key": "SECRET_KEY" // Replace with your actual secret
        }
    }).then(r=>{
      getProduct()
    })
  }

  const openDialog = (item) =>{
    dialogData.value = {...item}
    dialog.value = true
  }

  const updateProduct = () =>{
    axios.put(`https://zzez7yzcy0.execute-api.ap-southeast-2.amazonaws.com/items/${dialogData.value.id}`,
    dialogData.value,
    {
      headers: {
        "Content-Type": "application/json",
        "x-api-key": "SECRET_KEY" // Replace with your actual secret
      }
    }).then(r=>{
      let item = r.data.data;
      tableData.value = tableData.value.map((val) =>
        val.id === item.id ? item : val
      );
    })
  }

  onMounted(() => {
    getProduct()
  })
</script>