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
                    <i class="fa fa-align-justify"></i> Articulos
                    <button type="button" @click="abrirModal('articulo', 'registrar')" class="btn btn-secondary">
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
                                <input type="text" v-model="buscar" @keyup.enter="listarArticulo(1,buscar,criterio)" class="form-control" placeholder="Texto a buscar">
                                <button type="submit" @click="listarArticulo(1,buscar,criterio)" class="btn btn-primary"><i class="fa fa-search"></i> Buscar</button>
                            </div>
                        </div>
                    </div>
                    <table class="table table-bordered table-striped table-sm">
                        <thead>
                            <tr>
                                <th>Opciones</th>
                                <th>Código</th>
                                <th>Nombre</th>
                                <th>Categoría</th>
                                <th>Precio Venta</th>
                                <th>Stock </th>
                                <th>Estado</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="articulo in arrayArticulo" :key="articulo.id">
                                <td>
                                    <button type="button" @click="abrirModal('articulo', 'actualizar', articulo)" class="btn btn-warning btn-sm">
                                        <i class="icon-pencil"></i>
                                    </button> &nbsp;
                                    <template v-if="articulo.condicion">
                                        <button type="button" @click="desactivarArticulo(articulo.id)" class="btn btn-danger btn-sm">
                                            <i class="icon-trash"></i>
                                        </button>
                                    </template>   
                                    <template v-else>
                                        <button type="button" @click="activarArticulo(articulo.id)" class="btn btn-info btn-sm">
                                            <i class="icon-check"></i>
                                        </button>
                                    </template>   
                                </td>
                                <td v-text="articulo.codigo"></td>
                                <td v-text="articulo.nombre"></td>
                                <td v-text="articulo.nombre_categoria"></td>
                                <td v-text="articulo.precio_venta"></td>
                                <td v-text="articulo.stock"></td>
                                <td>
                                    <div v-if="articulo.condicion">
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
        <div class="modal fade" tabindex="-1" :class="{'mostrar' : modal }" role="dialog" 
            aria-labelledby="myModalLabel" style="display: none;" aria-hidden="true">
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
                                <label class="col-md-3 form-control-label" for="text-input">Categoria</label>
                                <div class="col-md-9">
                                    <select class="form-control" v-model="idcategoria">
                                        <option value="0" disabled>Seleccione</option>
                                        <option v-for="categoria in arrayCategoria" :key="categoria.id" 
                                            :value="categoria.id" v-text="categoria.nombre"></option>
                                    </select>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-md-3 form-control-label" for="text-input">Codigo</label>
                                <div class="col-md-9">
                                    <input type="text" v-model="codigo" class="form-control" placeholder="Ingrese codigo de barras">
                                    <barcode :value="codigo" :options="{ format: 'EAN-13'}">
                                        Generando código de barras.
                                    </barcode>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-md-3 form-control-label" for="text-input">Nombre</label>
                                <div class="col-md-9">
                                    <input type="text" v-model="nombre" class="form-control" placeholder="Nombre de articulo">
                                    <span class="help-block">(*) Ingrese el nombre de la articulo</span>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-md-3 form-control-label" for="email-input">Precio venta</label>
                                <div class="col-md-9" >
                                    <input type="number" v-model="precio_venta" class="form-control">
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-md-3 form-control-label" for="email-input">Stock</label>
                                <div class="col-md-9" >
                                    <input type="number" v-model="stock" class="form-control">
                                </div>
                            </div>
                            <div class="form-group row">
                                <label class="col-md-3 form-control-label" for="email-input">Descripcion</label>
                                <div class="col-md-9" >
                                    <input type="text" v-model="descripcion" class="form-control" placeholder="Descripcion del articulo">
                                </div>
                            </div>
                            <div v-show="errorArticulo" class="form-group row div-error">
                                <div class="text-center text-error" >
                                    <div v-for="error in errorMsjArticulo" :key="error" v-text="error"></div>
                                </div>
                            </div>
                        </form>
                    </div>
                    <div class="modal-footer">
                        <button type="button" @click="cerrarModal()" class="btn btn-secondary">Cerrar</button>
                        <button type="button" v-if="tipoAccion == 1" @click="registrarArticulo()" class="btn btn-primary">Guardar</button>
                        <button type="button" v-if="tipoAccion == 2" @click="actualizarArticulo()" class="btn btn-primary">Actualizar</button>
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
import VueBarcode from 'vue-barcode';
    export default {
        data (){
            return {
                articulo_id: 0,
                idcategoria: 0,
                nombre_categoria: '',
                codigo: '',
                nombre: '',
                precio_venta: 0,
                stock: 0,
                descripcion: '',
                arrayArticulo: [],
                modal: 0,
                tituloModal: '',
                tipoAccion: 0,
                errorArticulo: 0,
                errorMsjArticulo: [],
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
                buscar: '',
                arrayCategoria: []
            }
        },

        components: {
            'barcode': VueBarcode
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
            listarArticulo(page,buscar,criterio){
                let me = this;
                var url = '/articulo?page='+page+'&buscar='+buscar+'&criterio='+criterio
                axios.get(url)
                .then(function(r){
                    var respuesta = r.data;
                    me.arrayArticulo = respuesta.articulos.data; 
                    me.pagination = respuesta.pagination;
                })
                .catch(function(e){
                    console.log(e);
                });
            },

            registrarArticulo(){
                if(this.validarArticulo()){
                    return;
                }
                let me = this;
                axios.post('/articulo/registrar', {
                    'idcategoria':this.idcategoria,
                    'codigo': this.codigo,
                    'nombre': this.nombre,
                    'precio_venta': this.precio_venta,
                    'stock': this.stock,
                    'descripcion': this.descripcion
                })
                .then(function(r){
                    me.cerrarModal();
                    me.listarArticulo(1,'','nombre');
                })
                .catch(function(e){
                    console.log(e);
                })
            },

            actualizarArticulo(){
                if(this.validarArticulo()){
                    return;
                }
                let me = this;
                axios.put('/articulo/actualizar', {
                    'id': this.articulo_id,
                    'idcategoria':this.idcategoria,
                    'codigo': this.codigo,
                    'nombre': this.nombre,
                    'precio_venta': this.precio_venta,
                    'stock': this.stock,
                    'descripcion': this.descripcion
                })
                .then(function(r){
                    me.cerrarModal();
                    me.listarArticulo(1,'','nombre');
                })
                .catch(function(e){
                    console.log(e);
                })
            },
            
            desactivarArticulo(id){
                let me = this;
                swal({
                    title: '¿Esta seguro de desactivar el articulo?',
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Si, desactivar'
                    })
                    .then((result) => {
                        if (result.value) {
                            axios.put('/articulo/desactivar',{
                                'id': id
                            })
                            .then(function(r){
                                me.listarArticulo(1,'','nombre');
                            })
                            .catch(function(e){
                                console.log(e);
                            });
                            swal(
                            'Desactivado',
                            'Articulo desactivado correctamente.',
                            'success'
                            )
                        }
                })
            },

            activarArticulo(id){
                let me = this;
                swal({
                    title: '¿Esta seguro de activar el articulo?',
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Si, activar'
                    })
                    .then((result) => {
                        if (result.value) {
                            axios.put('/articulo/activar',{
                                'id': id
                            })
                            .then(function(r){
                                me.listarArticulo(1,'','nombre');
                            })
                            .catch(function(e){
                                console.log(e);
                            });
                            swal(
                            'Activado',
                            'Articulo activado correctamente.',
                            'success'
                            )
                        }
                })
            },

            validarArticulo(){
                this.errorArticulo = 0;
                this.errorMsjArticulo = [];
                if(this.idcategoria == 0) this.errorMsjArticulo.push("Seleccione una categoria");
                if(!this.nombre) this.errorMsjArticulo.push("El nombre del articulo no puede estar vacio.");
                if(!this.stock) this.errorMsjArticulo.push("El stock del articulo debe de ser un numero y no puede estar vacio");
                if(!this.precio_venta) this.errorMsjArticulo.push("El precio de venta del articulo debe de ser un numero y no puede estar vacio");
                
                if(this.errorMsjArticulo.length) this.errorArticulo = 1;
                
                return this.errorArticulo;
            },

            abrirModal(modelo, accion, data = []){
                switch(modelo)
                {
                    case "articulo":
                        switch(accion){
                            case "registrar":
                                this.modal = 1;
                                this.tipoAccion = 1;
                                this.tituloModal = 'Registrar Articulo';
                                this.idcategoria = 0;
                                this.nombre_categoria = '';
                                this.codigo = '';
                                this.nombre = '';
                                this.precio_venta = 0;
                                this.stock = 0;
                                this.descripcion = '';
                            break;

                            case "actualizar":
                                this.modal = 1;
                                this.tipoAccion = 2;
                                this.tituloModal = "Actualizar Articulo";
                                this.articulo_id = data.id;
                                this.idcategoria = data.idcategoria;
                                this.codigo = data.codigo;
                                this.nombre = data.nombre;
                                this.precio_venta = data.precio_venta;
                                this.stock = data.stock;
                                this.descripcion = data.descripcion;
                            break;
                        }
                        this.selectCategoria();
                }
            },
            
            cerrarModal(){
                this.modal = 0;
                this.tituloModal = '';
                this.idcategoria = 0;
                this.nombre_categoria = '';
                this.codigo = '';
                this.nombre = '';
                this.precio_venta = 0;
                this.stock = 0;
                this.descripcion = '';
                this.errorMsjArticulo = [];
                this.errorArticulo = 0;
            },

            cambiarPagina(page,buscar,criterio){
                let me = this;
                // actualiza la pagina actual
                me.pagination.current_page = page;
                me.listarArticulo(page,buscar,criterio);
            },

            selectCategoria(){
                let me = this;
                var url = '/categoria/selectCategoria';
                axios.get(url)
                .then(function(r){
                    var respuesta = r.data;
                    me.arrayCategoria = respuesta.categorias;
                })
                .catch(function(e){
                    console.log(e);
                });
            },
        },
        mounted(){
            this.listarArticulo(1,this.buscar,this.criterio);
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

