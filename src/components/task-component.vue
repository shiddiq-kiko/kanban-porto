<template>
          <!-- Task Page -->
          <section>
            <br><br>
            <center>
              <div class="container">
                <div class="card card-signup z-depth-0">
                  <div class="card-header text-left">
                    <a class="btn btn-success text-light" data-toggle="modal" data-target="#createCategory">Add Category</a>
                    <div class="modal fade" id="createCategory" tabindex="-1" role="dialog" aria-hidden="true">
                      <div class="modal-dialog" role="document">
                        <div class="modal-content" style="width: 50%;">
                          <div class="card card-signup z-depth-0 text-center">
                            <div class="card-body">
                              <form @submit.prevent="addCategory">
                                <div class="form-group">
                                    <label for="name">Category name</label>
                                    <input type="text" v-model="categoryName" name="categoryName" class="form-control" placeholder="input Category" required>
                                </div>
                                <button type="submit" class="btn btn-primary text-light">Add Category</button>
                              </form>
                            </div>
                          </div>
                        </div>
                      </div>
                    </div>
                    <a class="btn btn-warning text-primary" @click="backToOrg">List Organization</a>
                    <search class="text-right" @read='readTask'></search>
                  </div>
                  <div class="card-body text-left">
                    <div class="row my-4">
                      <div class="col mb-4" v-for="(board, index) in boards" :key="board.OrganizationId" >
                        <div class="card">
                          <div class="card-header bg-success text-light">
                            <label>{{board.name}}</label>
                          </div>
                          <div class="card-body text-left" style="padding: 0.2rem">
                            <CreateTask 
                            :id='board.id'
                            @read='readTask'
                            class="text-right text-dark">
                            </CreateTask>
                          </div>
                          <!-- <draggable v-model="boards"> -->
                              <transition-group>
                                  <div class="card-body drag-item" v-for="task in filterTask(board.id)" :key="task.id" >
                                    <div class="card text-left" >
                                      <div class="card-header">
                                        <label>{{task.title}}</label>
                                        <Update
                                        :taskName='task.title'
                                        :taskDescription='task.description'
                                        :id="task.id"
                                        @readTask='readTask'
                                        @readCategory='readCategory'>
                                        </Update>
                                      </div>
                                      <div class="card-body">
                                        <label>{{task.description}}</label>
                                      </div>
                                      <div class="card-footer text-center" style="padding: 0.1rem">
                                        <a class="btn text-warning" v-if="index != 0" @click="back(board.id, task.id)"><i class="fas fa-angle-double-left"></i></a>
                                        <a class="btn text-danger" @click='deleteTask(task.id)'><i class="fa fa-trash" aria-hidden="true"></i></a>
                                        <a class="btn text-success" v-if="index < boardLength-1" @click="next(board.id, task.id)"><i class="fas fa-angle-double-right"></i></a>
                                      </div>
                                    </div>
                                  </div>
                              </transition-group>
                          <!-- </draggable> -->
                          
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </center>
          </section>
</template>

<script>
import axios from 'axios'
import CreateTask from './subComponents/taskForm'
import Search from './subComponents/searchUser'
import Update from './subComponents/updateTask'
import draggable from 'vuedraggable'
export default {
  data () {
    return {
        boards : [],
        tasks : [],
        categoryName : '',
        taskName : '',
        taskDescription : '',
    }
  },
  components : {
    CreateTask,
    Search,
    draggable,
    Update
  },
  methods : {
    next(category, task){
      const CategoryId = category+1
      axios.patch(`https://quiet-depths-10928.herokuapp.com/tasks/${task}`, {
        CategoryId
      }, {
        headers : {
          token : localStorage.token,
          id : localStorage.id
        }
      })
      .then(({data}) => {
        this.count = this.count++ 
        this.readTask()
        this.readCategory()
      })
      .catch(err => {
        console.log(err)
      })
    },
    back(category, task){
      const CategoryId = category-1
      axios.patch(`https://quiet-depths-10928.herokuapp.com/tasks/${task}`, {
        CategoryId
      }, {
        headers : {
          token : localStorage.token,
          id : localStorage.id
        }
      })
      .then(({data}) => {
        this.count = this.count--
        this.readTask()
        this.readCategory()
      })
      .catch(err => {
        console.log(err)
      })
    },
    deleteTask(id){
      axios.delete(`https://quiet-depths-10928.herokuapp.com/tasks/${id}`, {
          headers : {
            token : localStorage.token
          }
      })
      .then(res => {
        this.readTask()
        this.readCategory()
      })
      .catch(err => {
        console.log(err)
      })
    },
    filterTask(id){
      const task = []
      this.tasks.forEach(el => {
        if(el.id === id){
          el.Tasks.forEach(els => {
            const temp = {
              id : els.id,
              title : els.title,
              description : els.description
            }
            task.push(temp)
          })
        }
      });
      return task
    },
    readTask(){
      axios.get('https://quiet-depths-10928.herokuapp.com/tasks', {
            headers : {
                token : localStorage.token,
                id : localStorage.id
            }
      })
      .then(({data}) => {
        this.tasks = data.result
        // console.log(this.tasks)
      })
      .catch(err => {
        console.log(err)
      })
    },
    backToOrg(){
      this.$emit('currentPage', 'mainPage')
    },
    addCategory(){
        axios.post('https://quiet-depths-10928.herokuapp.com/tasks/category',{
            category : this.categoryName
        },
        {
            headers : {
                token : localStorage.token,
                id : localStorage.id
            }
        })
        .then(({data}) => {
            // console.log(data)
            localStorage.removeItem('id')
            this.readCategory()
        })
        .catch(err => {
          console.log(err)
        })
    },
    readCategory(){
      console.log('masuuuk')
        axios.get('https://quiet-depths-10928.herokuapp.com/tasks/category', {
            headers : {
                token : localStorage.token,
                id : localStorage.id
            }
        })
        .then(({data}) => {
          // console.log(data.result)
            this.boards = data.result
        })
    }
  },
  created () {
    this.readCategory()
    this.readTask()
  },
  computed : {
    taskName(title){
      this.taskName = title
      return this.taskName
    },
    taskDescription(description){
      // console.log(this.boards.length)
      this.taskDescription = description
      return this.taskDescription
    }
  }
}
</script>

<style>

</style>