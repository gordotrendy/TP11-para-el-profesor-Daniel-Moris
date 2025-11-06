# TP11-para-el-profesor-Daniel-Moris
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>TP 11 DE JS, BOOTSTRAP Y LOCALSTORAGE</title>
    
    <!--esto es para utilizar el css de bootstrap-->

    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
    <style>
       body {
  --s: 160px; /* control the size*/
  --c1: #1c2130;
  --c2: #b3e099;
    
  --_c:5%,#0000 75%,var(--c1) 0;
  --_g:/var(--s) var(--s) conic-gradient(at var(--_c));
  --_l:/var(--s) var(--s) conic-gradient(at 50% var(--_c));
  background:
    0 calc(7*var(--s)/10) var(--_l),
    calc(var(--s)/2) calc(var(--s)/5) var(--_l),
    calc(var(--s)/5) 0 var(--_g),
    calc(7*var(--s)/10) calc(var(--s)/2) var(--_g),
    conic-gradient(at 90% 90%,var(--c1) 75%,var(--c2) 0)
    0 0/calc(var(--s)/2) calc(var(--s)/2);
}

        .note-card{
            transition: transform 0.2s;

        }
        .note-card:hover{
            transform: translateY(-5px);

        }
        .empty-state{
            text-align: center;
            color: #6c757d;
            padding: 2rem;

        }
        h1{
            color:rgb(255, 255, 255);
    
        }
        h3{
            color: white;
        }
        p{
            color:white;
        }
    </style>
</head>
<body>
    <div class="container mt-4">
        <h1 class="text-center mb-4 ">Gestor de Notas</h1>

        <!--Formulario para agregar/editar notas-->
        <div class="card p-3 mb-4 shadow-sm">
            <form id="noteForm">
                <input type="hidden" id="noteId">
                <div class="mb-3 border border-primary">
                    <input type="text" id="titulo" class="form-control" placeholder="Titulo de la nota"required>
                    
                </div>
                <div class="mb-3">
                    <textarea id="contenido" class="form-control border border-primary" placeholder="Escriba su nota aqui :D" rows="4" required></textarea>
                </div>
                <div class="d-grid gap-2 d-md-flex justify-content-md-end ">
                    <button type="button" id="cancelEdit" class="btn btn-secondary me-md-2" style="display: none;">Cancelar</button>
                    <button type="submit" id="guardar" class="btn btn-success">Guardar Nota</button>
                </div>
            </form>
        </div>
        <hr>
        <!--Seccion para mostrar las notas-->
        <div id="notas" class="row">
            <!--aca se cargan las notas rey-->
        </div>
        <!--boton para eliminar todas las notas-->
        <div class="text-center mt-4">
            <button id="borrarTodo" class="btn btn-danger" style="display: none;">Eliminar todas las notas</button>
        </div>
    </div>

    <!--aqui comienzan las funciones de script-->
    <script>
        class GestorNotas {
            constructor(){
                this.notas=this.obtenerNotas();
                this.modoEdicion=false;
                this.notaEditando=null;
                this.inicializarEventos();
                this.mostrarNotas();
            }
            //obtengo las notas del localstorage//
            obtenerNotas(){
                const notasGuardadas=localStorage.getItem('notas');
                return notasGuardadas ? JSON.parse(notasGuardadas) : [];
            }
            //guardo las notas en el localstorage//
            guardarNotaLocal(){
                localStorage.setItem('notas', JSON.stringify(this.notas));
            }
            //iniciar event listeners//
            inicializarEventos(){
                document.getElementById('noteForm').addEventListener('submit', (e) => this.guardarNota(e));
                document.getElementById('borrarTodo').addEventListener('click', () =>this.borrarTodas());
                document.getElementById('cancelEdit').addEventListener('click', () =>this.cancelarEdicion());
            }
            //guardar nota nueva o actualizar existente//
            guardarNota(e){
                e.preventDefault();

                const titulo=document.getElementById('titulo').value.trim();
                const contenido=document.getElementById('contenido').value.trim();
                if (!titulo || !contenido)
                {
                    alert("Por favor, complete el titulo como la nota.");
                    return;
                }
                if (this.modoEdicion)
                {
                    this.notaEditando.titulo=titulo;
                    this.notaEditando.contenido=contenido;
                }
                else
                {
                    //hago que lo cree//
                    const nuevaNota={
                        id:Date.now(),
                        titulo: titulo,
                        contenido: contenido,
                        fecha: new Date().toLocaleDateString()
                    };
                    this.notas.push(nuevaNota);
                }
                this.guardarNotaLocal();
                this.mostrarNotas();
                this.limpiarFormulario();
                this.cancelarEdicion();
            }
            //mostrar todas las notas :D//
            mostrarNotas(){
                const contenedorNotas=document.getElementById('notas');
                const botonBorrarTodo=document.getElementById('borrarTodo');

                if (this.notas.length===0)
                {
                    contenedorNotas.innerHTML= `
                    <div class="empty-state">
                        <h3>No hay notas</h3>
                        <p>Dale agrega una nota</p>
                    </div>
                    `;
                    
                    botonBorrarTodo.style.display='none';
                    return;
                
                    
                }
                botonBorrarTodo.style.display='block';
                contenedorNotas.innerHTML='';
                this.notas.forEach(nota => {
                    const notaElement = document.createElement('div');
                    notaElement.className = 'col-md-6 col-lg-4 mb-3';
                    notaElement.innerHTML = `
                        <div class="card note-card h-100 shadow">
                            <div class="card-body">
                                <h5 class="card-title">${nota.titulo}</h5>
                                <p class="card-text">${nota.contenido}</p>
                                <small class="text-muted">Creada: ${nota.fecha}</small>
                            </div>
                            <div class="card-footer bg-transparent">
                                <div class="btn-group w-100">
                                    <button class="btn btn-outline-primary btn-sm" onclick="gestorNotas.editarNota(${nota.id})">
                                        Editar
                                    </button>
                                    <button class="btn btn-outline-danger btn-sm" onclick="gestorNotas.eliminarNota(${nota.id})">
                                        Eliminar
                                    </button>
                                </div>
                            </div>
                        </div>
                    `;
                    contenedorNotas.appendChild(notaElement);
                });
            }
            //editar una nota existente//
            editarNota(id){
                const nota=this.notas.find(n=>n.id===id);
                if (nota){
                    this.modoEdicion=true;
                    this.notaEditando=nota;

                    document.getElementById('noteId').value=id;
                    document.getElementById('titulo').value=nota.titulo;
                    document.getElementById('contenido').value=nota.contenido;
                    document.getElementById('guardar').textContent= 'Actualizar Nota';
                    document.getElementById('cancelEdit').style.display='block';

                    document.getElementById('titulo').focus();
                }
            }
            //eliminar una nota individual//
            eliminarNota(id){
                if (confirm('¿Seguro queres eliminar la nota?'))
            {
                this.notas=this.notas.filter(nota=>nota.id !==id);
                this.guardarNotaLocal();
                this.mostrarNotas();
            }

            }
            //borrar todas las notas//
            borrarTodas()
            {
                if (this.notas.length===0) return;

                if (confirm('¿Seguro queres eliminar todas las notas? Esto no se podra deshacer.'))
                {
                    this.notas=[];
                    this.guardarNotaLocal();
                    this.mostrarNotas();

                }
            }
            //cancelar modo edicion//
            cancelarEdicion()
            {
                this.modoEdicion=false;
                this.notaEditando=null;
                this.limpiarFormulario();
                document.getElementById('guardar').textContent='Guardar Nota';
                document.getElementById('cancelEdit').style.display='none';
            }
            //limpiar el formulario//
            limpiarFormulario()
            {
                document.getElementById('noteForm').reset();
                document.getElementById('noteId').value='';
            }
        
        }
        //inicializar la aplicacion cuando el DOM este listo//
        document.addEventListener('DOMContentLoaded',()=>{
            window.gestorNotas=new GestorNotas();
        });
        
    </script>
    
</body>
</html>
