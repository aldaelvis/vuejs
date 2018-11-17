<template>
    <main class="main">
        <!-- Breadcrumb -->
        <ol class="breadcrumb">
            <li class="breadcrumb-item"><a href="#">Escritorio</a></li>
        </ol>
        <div class="container-fluid">
            <!-- Ejemplo de tabla Listado -->
            <div class="card">
                <div class="card-header">
                    <i class="fa fa-align-justify"></i> Categorías
                    <button type="button" @click="abrirModal('categoria', 'registrar')" class="btn btn-secondary">
                        <i class="icon-plus"></i>&nbsp;Nuevo
                    </button>
                </div>
                <div class="card-body">
                    <div class="form-group row">
                        <div class="col-md-6">
                            <div class="input-group">
                                <select class="form-control col-md-3" v-model="criterio">
                                    <option value="nombre">Nombre</option>
                                    <option value="descripcion">Descripción</option>
                                </select>
                                <input type="text" v-model="buscar" @keyup.enter="listarCategoria(1,buscar,criterio)" class="form-control" placeholder="Texto a buscar">
                                <button type="submit" @click="listarCategoria(1,buscar,criterio)" class="btn btn-primary"><i class="fa fa-search"></i> Buscar</button>
                            </div>
                        </div>
                    </div>
                    <table class="table table-bordered table-striped table-sm">
                        <thead>
                            <tr>
                                <th>Opciones</th>
                                <th>Nombre</th>
                                <th>Descripción</th>
                                <th>Estado</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="categoria in arrayCategoria" :key="categoria.id">
                                <td>
                                    <button type="button" @click="abrirModal('categoria', 'actualizar', categoria)" class="btn btn-warning btn-sm">
                                        <i class="icon-pencil"></i>
                                    </button> &nbsp;
                                    <template v-if="categoria.condicion">
                                        <button type="button" @click="desactivarCategoria(categoria.id)" class="btn btn-danger btn-sm">
                                            <i class="icon-trash"></i>
                                        </button>
                                    </template>   
                                    <template v-else>
                                        <button type="button" @click="activarCategoria(categoria.id)" class="btn btn-info btn-sm">
                                            <i class="icon-check"></i>
                                        </button>
                                    </template>   
                                </td>
                                <td v-text="categoria.nombre">Equipos</td>
                                <td v-text="categoria.descripcion">Dispositivos electrónicos</td>
                                <td>
                                    <div v-if="categoria.condicion">
                                        <span class="badge badge-success">Activo</span>
                                    </div>
                                    <div v-else>
                                        <span class="badge badge-danger">Desactivo</span>
                                    </div>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                    <nav>
                        <ul class="pagination">
                            <li class="page-item" v-if="pagination.current_page > 1">
                                <a class="page-link" href="#" 
                                    @click.prevent="cambiarPagina(pagination.current_page-1,buscar,criterio)">Ant</a>
                            </li>
                            <li class="page-item" v-for="page in pageNumber" :key="page" :class="[page == isActive ? 'active' : '']">
                                <a class="page-link" href="#" @click.prevent="cambiarPagina(page,buscar,criterio)" v-text="page"></a>
                            </li>
                            <li class="page-item" v-if="pagination.current_page < pagination.last_page">
                                <a class="page-link" href="#" @click.prevent="cambiarPagina(pagination.current_page + 1,buscar,criterio)">Sig</a>
                            </li>
                        </ul>
                    </nav>
                </div>
            </div>
            <!-- Fin ejemplo de tabla Listado -->
        </div>
        <!--Inicio del modal agregar/actualizar-->
        <div class="modal fade" tabindex="-1" :class="{'mostrar' : modal }" role="dialog" aria-labelledby="myModalLabel" style="display: none;" aria-hidden="true">
            <div class="modal-dialog modal-primary modal-lg" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h4 class="modal-title" v-text="tituloModal"></h4>
                        <button type="button" class="close" @click="cerrarModal()" aria-label="Close">
                            <span aria-hidden="true">×</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <form action="" method="post" enctype="multipart/form-data" class="form-horizontal">
                            <div class="form-group row">
                                <label class="col-md-3 form-control-label" for="text-input">Nombre</label>
                                <div class="col-md-9">
                                    <input type="text" v-model="nombre" class="form-control" placeholder="Descripcion de categoría">
                                    <span class="help-block">(*) Ingrese el nombre de la categoría</span>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-md-3 form-control-label" for="email-input">Descripción</label>
                                <div class="col-md-9" >
                                    <input type="text" v-model="descripcion" class="form-control" placeholder="Enter Email">
                                </div>
                            </div>
                            <div v-show="errorCategoria" class="form-group row div-error">
                                <div class="text-center text-error" >
                                    <div v-for="error in errorMsjCategoria" :key="error" v-text="error"></div>
                                </div>
                            </div>
                        </form>
                    </div>
                    <div class="modal-footer">
                        <button type="button" @click="cerrarModal()" class="btn btn-secondary">Cerrar</button>
                        <button type="button" v-if="tipoAccion == 1" @click="registrarCategoria()" class="btn btn-primary">Guardar</button>
                        <button type="button" v-if="tipoAccion == 2" @click="actualizarCategoria()" class="btn btn-primary">Actualizar</button>
                    </div>
                </div>
                <!-- /.modal-content -->
            </div>
            <!-- /.modal-dialog -->
        </div>
        <!--Fin del modal-->
    </main>
</template>

<script>
    export default {
        data (){
            return {
                categoria_id: 0,
                nombre: '',
                descripcion: '',
                arrayCategoria: [],
                modal: 0,
                tituloModal: '',
                tipoAccion: 0,
                errorCategoria: 0,
                errorMsjCategoria: [],
                // recuperamos los nombres de paginacion {laravel}
                pagination: {
                    'total': 0,
                    'current_page': 0,
                    'per_page': 0,
                    'last_page': 0,
                    'from': 0,
                    'to': 0 
                },
                offset: 3,
                criterio: 'nombre',
                buscar: ''
            }
        },

        computed: {
            isActive: function(){
                return this.pagination.current_page;
            },
            // calculamos los elementos de la paginacion
            pageNumber: function(){
                if(!this.pagination.to){
                    return [];
                }
                var from = this.pagination.current_page - this.offset;
                if(from < 1) {
                    from = 1;
                }
                var to = from + (this.offset * 2);
                if(to >= this.pagination.last_page){
                    to = this.pagination.last_page;
                }
                var pagesArray = [];
                while(from <= to){
                    pagesArray.push(from);
                    from++;
                }
                return pagesArray;
            }
        },

        methods: {
            listarCategoria(page,buscar,criterio){
                let me = this;
                var url = '/categoria?page='+page+'&buscar='+buscar+'&criterio='+criterio
                axios.get(url)
                .then(function(r){
                    var respuesta = r.data;
                    me.arrayCategoria = respuesta.categorias.data; 
                    me.pagination = respuesta.pagination;
                })
                .catch(function(e){
                    console.log(e);
                });
            },

            registrarCategoria(){
                if(this.validarCategoria()){
                    return;
                }
                let me = this;
                axios.post('/categoria/registrar', {
                    'nombre': this.nombre,
                    'descripcion': this.descripcion
                })
                .then(function(r){
                    me.cerrarModal();
                    me.listarCategoria(1,'','nombre');
                })
                .catch(function(e){
                    console.log(e);
                })
            },

            actualizarCategoria(){
                if(this.validarCategoria()){
                    return;
                }
                let me = this;
                axios.put('/categoria/actualizar', {
                    'id': this.categoria_id,
                    'nombre': this.nombre,
                    'descripcion': this.descripcion
                })
                .then(function(r){
                    me.cerrarModal();
                    me.listarCategoria(1,'','nombre');
                })
                .catch(function(e){
                    console.log(e);
                })
            },
            
            desactivarCategoria(id){
                let me = this;
                swal({
                    title: '¿Esta seguro de desactivar la categoria?',
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Si, desactivar'
                    })
                    .then((result) => {
                        if (result.value) {
                            axios.put('/categoria/desactivar',{
                                'id': id
                            })
                            .then(function(r){
                                me.listarCategoria(1,'','nombre');
                            })
                            .catch(function(e){
                                console.log(e);
                            });
                            swal(
                            'Desactivado',
                            'Categoria desactivada correctamente.',
                            'success'
                            )
                        }
                })
            },

            activarCategoria(id){
                let me = this;
                swal({
                    title: '¿Esta seguro de activar la categoria?',
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Si, activar'
                    })
                    .then((result) => {
                        if (result.value) {
                            axios.put('/categoria/activar',{
                                'id': id
                            })
                            .then(function(r){
                                me.listarCategoria(1,'','nombre');
                            })
                            .catch(function(e){
                                console.log(e);
                            });
                            swal(
                            'Activado',
                            'Categoria activado correctamente.',
                            'success'
                            )
                        }
                })
            },

            validarCategoria(){
                this.errorCategoria = 0;
                this.errorMsjCategoria = [];
                if(!this.nombre) this.errorMsjCategoria.push("El nombre de la categoria no puede estar vacio.");
                if(this.errorMsjCategoria.length) this.errorCategoria = 1;
                
                return this.errorCategoria;
            },

            abrirModal(modelo, accion, data = []){
                switch(modelo)
                {
                    case "categoria":
                        switch(accion){
                            case "registrar":
                                this.modal = 1;
                                this.tipoAccion = 1;
                                this.tituloModal = 'Registrar Categoria';
                                this.nombre = '';
                                this.descripcion = '';
                            break;

                            case "actualizar":
                                this.modal = 1;
                                this.tipoAccion = 2;
                                this.tituloModal = "Actualizar Categoria";
                                this.categoria_id = data.id;
                                this.nombre = data.nombre;
                                this.descripcion = data.descripcion;
                            break;
                        }
                }
            },
            
            cerrarModal(){
                this.modal = 0;
                this.tituloModal = '';
                this.nombre = '';
                this.descripcion = '';
                this.errorMsjCategoria = [];
                this.errorCategoria = 0;
            },

            cambiarPagina(page,buscar,criterio){
                let me = this;
                // actualiza la pagina actual
                me.pagination.current_page = page;
                me.listarCategoria(page,buscar,criterio);
            }
        },
        mounted(){
            this.listarCategoria(1,this.buscar,this.criterio);
        }
    }
</script>
<style>
    .modal-content{
        width: 100% !important;
        position: absolute !important; 
    }
    .mostrar{
        display: list-item !important;
        opacity: 1 !important;
        position: absolute !important;;
        background: #3c29297a !important;;
    }
    .div-error{
        display: flex;
        justify-content: center;
    }
    .text-error{
        color: red;
        font-weight: bold;
    }
</style>

