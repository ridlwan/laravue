<template>
    <div class="row">
        <div class="col-md-8">
            <div class="row">
                <div v-for="product in products" v-bind:key="product.id" class="card card-body col-md-4">
                    <h4>{{ product.name }}</h4>
                    <p>{{ product.description }}</p>
                    <div class="row">
                        <div class="col-md-6">Stock {{ product.amount }}</div>
                        <div class="col-md-6 text-right">Rp. {{ product.price }}</div>
                    </div>
                    <p class="text-right mt-2">
                        <button @click="editProduct(product)" class="btn btn-warning">Edit</button>
                        <button @click="deleteProduct(product.id)" class="btn btn-danger">Delete</button>
                    </p>
                </div>
            </div>
            <div class="row mt-2">
                <div class="col-md-8">
                    <nav>
                        <ul class="pagination">
                            <li v-bind:class="{disabled:!pagination.first_link}" class="page-item"><a href="#" @click="viewProduct(pagination.first_link)" class="page-link">&laquo;</a></li>
                            <li v-bind:class="{disabled:!pagination.prev_link}" class="page-item"><a href="#" @click="viewProduct(pagination.prev_link)" class="page-link">&lt;</a></li>
                            <li v-for="n in pagination.pagesArray" v-bind:key="n" v-bind:class="{active: pagination.current_page == n}" class="page-item"><a href="#" @click="viewProduct(pagination.path_page + n)" class="page-link">{{ n }}</a></li>
                            <li v-bind:class="{disabled:!pagination.next_link}" class="page-item"><a href="#" @click="viewProduct(pagination.next_link)" class="page-link">&gt;</a></li>
                            <li v-bind:class="{disabled:!pagination.last_link}" class="page-item"><a href="#" @click="viewProduct(pagination.last_link)" class="page-link">&raquo;</a></li>
                        </ul>
                    </nav>
                </div>
                <div class="col-md-4">
                    Total Page: {{ pagination.last_page }}
                </div>
            </div>
        </div>
        <div class="col-md-4">
            <form>
                <div class="form-group">
                    <label>Name</label>
                    <input type="text" class="form-control" v-model="product.name">
                    <span v-if="errors.name" class="error">{{ errors.name[0] }}</span>
                </div>
                <div class="form-group">
                    <label>Description</label>
                    <textarea type="text" class="form-control" v-model="product.description"></textarea>
                    <span v-if="errors.description" class="error">{{ errors.description[0] }}</span>
                </div>
                <div class="form-group">
                    <label>Price</label>
                    <input type="number" class="form-control" v-model="product.price">
                    <span v-if="errors.price" class="error">{{ errors.price[0] }}</span>
                </div>
                <div class="form-group">
                    <label>Amount</label>
                    <input type="number" class="form-control" v-model="product.amount">
                    <span v-if="errors.amount" class="error">{{ errors.amount[0] }}</span>
                </div>
                <button v-if="add" @click.prevent="addProduct()" class="btn btn-primary">Add Product</button>
                <button v-if="edit" @click.prevent="updateProduct(product.id)" class="btn btn-warning">Edit Product</button>
                <button @click.prevent="productClear()" class="btn btn-info">Clear</button>
            </form>
        </div>
    </div>    
</template>

<style>
    .product-item{
        width: 350px;
        float: left;
    }
</style>

<script>
    export default {
        data() {
            return {
                products: [],
                product: {
                    id: '',
                    name: '',
                    description: '',
                    price: '',
                    amount: ''
                },
                add: true,
                edit: false,
                pagination: {},
                errors: [],
                offset: 3
            }
        },
        created() {
            this.viewProduct();
        },
        methods: {
            viewProduct(paginate) {
                paginate = paginate || '/api/products';

                axios.get(paginate)
                .then(response => {
                    this.products = response.data.data;

                    if (!response.data.meta.to) {
                        return [];
                    }
                    
                    var from = response.data.meta.current_page - this.offset;
                    if (from < 1) {
                        from = 1;
                    }
                    
                    var to = from + (this.offset * 2);
                    if (to >= response.data.meta.last_page) {
                        to = response.data.meta.last_page;
                    }
                    
                    var pagesArray = [];
                    while (from <= to) {
                        pagesArray.push(from);
                        from++;
                    }

                    this.pagination = {
                        current_page: response.data.meta.current_page,
                        last_page: response.data.meta.last_page,
                        pagesArray: pagesArray,
                        from_page: response.data.meta.from,
                        to_page: response.data.meta.to,
                        total_page: response.data.meta.total,
                        path_page: response.data.meta.path+'?page=',
                        first_link: response.data.links.first,
                        last_link: response.data.links.last,
                        prev_link: response.data.links.prev,
                        next_link: response.data.links.next
                    };
                })
                .catch(error => {
                    console.log(error)
                })
            },
            addProduct() {
                axios.post('/api/products', {
                    name: this.product.name,
                    description: this.product.description,
                    price: this.product.price,
                    amount: this.product.amount
                })
                .then(response => {
                    swal('Successful!', 'Product has been added', 'success');
                    console.log(response)

                    this.product.name = '';
                    this.product.description = '';
                    this.product.price = '';
                    this.product.amount = '';
                    this.viewProduct();

                    this.errors = []
                })
                .catch(error => {
                    if (error.response.status == 422) {
                        this.errors = error.response.data.errors
                    } else {
                        swal('Failed!', 'Product fail to add', 'error');
                    }
                })
            },
            editProduct(pro) {
                this.add = false;
                this.edit = true;
                this.product.id = pro.id;
                this.product.name = pro.name;
                this.product.description = pro.description;
                this.product.price = pro.price;
                this.product.amount = pro.amount;
            },
            updateProduct(id) {
                axios.put(`/api/products/${id}`, {
                    name: this.product.name,
                    description: this.product.description,
                    price: this.product.price,
                    amount: this.product.amount
                })
                .then(response => {
                    swal('Successful!', 'Product has been updated', 'success');
                    this.add = true;
                    this.edit = false;
                    this.product.name = '';
                    this.product.description = '';
                    this.product.price = '';
                    this.product.amount = '';
                    this.viewProduct();

                    this.errors = []
                })
                .catch(error => {
                    if (error.response.status == 422) {
                        this.errors = error.response.data.errors
                    } else {
                        swal('Failed!', 'Product fail to update', 'error');
                    }
                })
            },
            deleteProduct(id) {
                swal({
                    title: 'Are you sure?',
                    text: 'Product will be delete',
                    icon: 'warning',
                    buttons: true,
                    dangerMode: true
                })
                .then((willdelete) => {
                    if(willdelete) {
                        axios.delete(`api/products/${id}`)
                        .then(response => {
                            swal('Successful!', 'Product has been deleted', 'success');
                            this.viewProduct();
                        })
                        .catch(error => {
                            console.log(error)
                            swal('Failed!', 'Product fail to delete', 'error');
                        })
                    }
                })
            },
            productClear() {
                this.add = true;
                this.edit = false;
                this.product.id = '';
                this.product.name = '';
                this.product.description = '';
                this.product.price = '';
                this.product.amount = '';
            }
        },   
    }
</script>
