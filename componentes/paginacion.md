## Paginación de registros (php y  mysql)

~~~ php
$tamaño_pagina = 5;
$empezar_desde = ($pagina - 1 ) * $tamaño_pagina;
$datos = array();

$query = "SELECT * FROM Tclasificados";
$stmt = Conexion::conectar()->prepare($query);
$stmt->execute(array());
$num_filas = $stmt->rowCount();
$total_paginas = (int) ceil($num_filas / $tamaño_pagina);
$from = ($pagina * $tamaño_pagina) - $tamaño_pagina;
$to =  ($pagina * $tamaño_pagina);


$query2 = "SELECT * FROM Tclasificados LIMIT $empezar_desde, $tamaño_pagina";
$stmt = Conexion::conectar()->prepare($query2);
// $stmt2 -> bindParam(':buscar', $buscar, PDO::PARAM_STR);
$stmt->execute();
$datos = $stmt->fetchAll();


return $total_array = array(
	"datos"=> $datos,
	"paginacion" => array(
		"total" => $num_filas,
		"current_page" => (int) $pagina,
		"per_page" => $tamaño_pagina,
		"last_page" => $total_paginas,
		"from" => $from,
		"to" => $to,
	)
);
~~~

## Paginación en VueJs

~~~ javascript
new Vue({
    el: '#app1',
    data: {
      arrayClasificado: [],
      buscar: '',
      paginacion: {
          'total': 0,
          'current_page': 0,
          'per_page': 0,
          'last_page': 0,
          'from': 0,
          'to': 0
      },
      offset: 3,
    },
    computed: {
      isActive: function(){
        return this.paginacion.current_page;
      },

      pageNumber: function(){
        if(!this.paginacion.to){
          return [];
        }
        var from = this.paginacion.current_page - this.offset;
        if(from < 1) {
            from = 1;
        }
        var to = from + (this.offset * 2);
        if(to >= this.paginacion.last_page){
            to = this.paginacion.last_page;
        }
        var pagesArray = [];
        while(from <= to){
            pagesArray.push(from);
            from++;
        }
        return pagesArray;
      },

    },

    methods: {
        paginacionClasificados: function(page, buscar){
          let me = this;
          axios.get('vista/modulos/ajax/ajax.php?op=paginacion&pagina=' + page)
          .then(function(response){
            var respuesta = response.data;
            me.paginacion = respuesta.paginacion;
            me.arrayClasificado = respuesta.datos;
          }).catch(function(error){
            console.log(error);
          })
        },

        cambiarPagina: function(page){
            let me = this;
            //actualiza la pagina actual
            me.paginacion.current_page = page;
            me.paginacionClasificados(page);
        },
    mounted(){
      // this.listarClasificadosUser();
      this.paginacionClasificados(1);
    },


  });
~~~
## Paginación en la vista

~~~ html
<ul class="paginacion">
    <li class="page-item" v-if="paginacion.current_page > 1">
	<a class="page-link" href="#"
	@click.prevent="cambiarPagina(paginacion.current_page - 1, buscar)">Ant</a>
    </li>
    <li class="page-item" v-for="page in pageNumber" :key="page"
	:class="[page == isActive ? 'active' : '']">
	<a class="page-link" href="#" @click.prevent="cambiarPagina(page, buscar)" v-text="page"></a>
    </li>
    <li class="page-item" v-if="paginacion.current_page < paginacion.last_page">
	<a class="page-link" href="#"
	@click.prevent="cambiarPagina(paginacion.current_page +1, buscar)">Sig</a>
    </li>
</ul>
~~~

## Estilos css para la paginación
~~~ css
.paginacion {
    display: flex;
    padding-left: 0;
    list-style: none;
    border-radius: .25rem;
    justify-content: center;
}

.page-item::after, ::before{
    box-sizing: border-box;
}
.page-item:first-child .page-link{
    margin-left: 0;
    border-top-left-radius: .25rem;
    border-bottom-left-radius: .25rem;
}
.page-link{
    position: relative;
    display: block;
    padding: .2rem .65rem;
    margin-left: -1px;
    line-height: 1.25;
    color: #007bff;
    background-color: #fff;
    border: 1px solid #dee2e6;
}

.page-item.active .page-link{
    z-index: 1;
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
}
~~~
