<template>
    <div class="container">
        <div class="row">
            <div class="col-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Users</h3>

                        <div class="card-tools">
                            <button class="btn btn-sm btn-primary" @click="newModal">Add
                                user <i class="fa fa-user-plus"></i></button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Type</th>
                                <th>Action</th>
                            </tr>
                            <tbody>
                            <tr v-for="user in users" :key="user.id">
                                <td>{{user.id}}</td>
                                <td>{{user.name}}</td>
                                <td>{{user.email}}</td>
                                <td><span class="tag tag-danger">{{user.type}}</span></td>
                                <td>
                                    <a href="#" @click="editUser(user)">
                                        <i class="fa fa-edit"></i>
                                    </a>
                                    /
                                    <a href="#" @click="deleteUser(user.id)">
                                        <i class="fa fa-trash"></i>
                                    </a>
                                </td>
                            </tr>
                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                </div>
                <!-- /.card -->
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="addNewUser" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel"
             aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" v-show="editmode" id="exampleModalLabel">Update user</h5>
                        <h5 class="modal-title" v-show="!editmode" id="exampleModalLabel">Add new user</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editmode ? updateUser() : createUser()">
                        <div class="modal-body">
                            <div class="form-group">
                                <input v-model="form.name" type="text" name="name" placeholder="Name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                            </div>

                            <div class="form-group">
                                <input v-model="form.email" type="email" name="email" placeholder="Email address"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                            </div>

                            <div class="form-group">
                                <input v-model="form.password" type="password" name="password" placeholder="Password"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>

                            <div class="form-group">
                                <textarea v-model="form.bio" name="bio" id="bio" placeholder="bio" rows="5"
                                          class="form-control"
                                          :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>

                            <div class="form-group">
                                <select v-model="form.type" name="type" id="type" placeholder="type"
                                        class="form-control"
                                        :class="{ 'is-invalid': form.errors.has('type') }">
                                    <option value="" disabled selected>Please select</option>
                                    <option value="admin">admin</option>
                                    <option value="user">standard user</option>
                                    <option value="author">author</option>

                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>


                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary btn-sm" data-dismiss="modal">Close</button>
                            <button type="submit" v-show="editmode" class="btn btn-primary btn-sm">Update</button>
                            <button type="submit" v-show="!editmode" class="btn btn-primary btn-sm">Create</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>


    </div>
</template>

<script>
    export default {
        data() {

            return {
                editmode: false,
                users: {},
                form: new Form({
                    id:'',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },
        methods: {
            createUser() {
                this.$Progress.start();
                this.form.post('api/user').then(() => {
                    this.$Progress.finish();
                    Fire.$emit('LoadAllUsers');
                    $('#addNewUser').modal('hide');
                    toast({
                        type: 'success',
                        title: 'User created successfully'
                    });
                }).catch(() => {
                    this.$Progress.fail();
                });
            },
            updateUser() {
                this.$Progress.start();
                this.form.put('api/user/' + this.form.id).then((response)=>{
                    Fire.$emit('LoadAllUsers');
                    $('#addNewUser').modal('hide');
                    toast({
                        type: 'success',
                        title: 'User Updated successfully'
                    });
                    this.$Progress.finish();
                }).catch((error)=>{
                    this.$Progress.fail();
                })
            },
            loadUsers() {
                axios.get('api/user').then(({data}) => (this.users = data.data));
            },
            editUser(user) {
                this.editmode = true;
                this.editModal(user);
            },
            deleteUser(userId) {
                swal({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if (result.value) {
                        this.$Progress.start();
                        this.form.delete('api/user/' + userId).then((response) => {
                            this.$Progress.finish();
                            swal(
                                'Deleted!',
                                response.data.message,
                                'success'
                            );
                            Fire.$emit('LoadAllUsers');
                        }).catch((error) => {
                            this.$Progress.fail();
                            swal(
                                'Deleted!',
                                "something went wrong",
                                'success'
                            )
                        });
                    }
                });


            },
            newModal() {
                this.editmode = false;
                this.form.reset();
                $('#addNewUser').modal('show');
            },
            editModal(user) {
                this.form.reset();
                this.form.clear();
                $('#addNewUser').modal('show');
                this.form.fill(user);
            }
        },
        created() {
            this.loadUsers();
            Fire.$on('LoadAllUsers', () => {
                this.loadUsers();
            });
        }
    }
</script>
