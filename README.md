# search/imagen
//buscador de imagenes gratis para un pagina web

const formulario = document.querySelector('#formulario');
const resultado = document.querySelector('#resultado');
const paginacionDiv= document.querySelector('#paginacion');

const registrosPorPagina = 40;
let totalPaginas;
let iterador;
let paginaActual = 1;

window.onload = () => {
    // Validar formulario;
    formulario.addEventListener('submit', validarFormulario);

}


function validarFormulario(e){
    e.preventDefault();

    const terminoBusqueda = document.querySelector('#termino').value;

        if(terminoBusqueda ===''){
            mostrarAlerta('Agrega un termino de busqueda');
            return;
        }
        // Seleccionamos terminoBusqueda para tener el value y ese mismo utilizarlo
        buscarImagenes();
   

}

function mostrarAlerta(mensaje){
    // Elejimos una clase de nuestro error y si no existe, agregamos la alerta, si existe ya no la agregamos
    const existeAlerta = document.querySelector('.bg-red-100')
    // !existeAlerta = significa si no existe... 
    if(!existeAlerta){

        const alerta = document.createElement('p');
        alerta.classList.add('bg-red-100','border-red-400','text-red-700','px-4','py-3','rounded','max-w-lg','mx-auto','mt-6','text-center');
    
        alerta.innerHTML = `<strong class="font-bold">Error!</strong>
        <span class="block sm:inline">${mensaje}</span>`;
        // Agregamos la alerta
        formulario.appendChild(alerta);
    
        setTimeout(()=>{
            alerta.remove();
        },3000);

    }
    
}


function buscarImagenes(){
    const termino = document.querySelector('#termino').value;
    const key = '20339195-cb567e902ce2949aa48940850';
    const url = `https://pixabay.com/api/?key=${key}&q=${termino}&per_page=${registrosPorPagina}&page=${paginaActual}`;

    fetch(url).then(respuesta => respuesta.json()).then(resultado => {
        totalPaginas = calcularPaginas(resultado.totalHits);
        mostrarImagenes(resultado.hits);
    })
}
// Generador que va a registrar la cantidad de elementos de acuerdo a las paginas
// un generador se realiza por function y aterisco...

function *crearPaginador(total){
    for(let i= 1; i<= total; i++){
        yield i;
    }
}

function calcularPaginas(total){
    return parseInt(Math.ceil(total / registrosPorPagina));
}

function mostrarImagenes(imagenes){
    // Miestras haya algo ahi...
    while(resultado.firstChild){
        resultado.removeChild(resultado.firstChild);
    }
    // Iterar sobre el arreglo de images y construir el html;
    imagenes.forEach( imagen => {
        const {previewURL, likes, views,largeImageURL} = imagen;
        resultado.innerHTML += `
            <div class="w-1/2 md:w-1/3 lg:w-1/4 p-3 mb-4">
                <div class="bg-white">
                     <img class="w-full" src= "${previewURL}">

                     <div class="p-4">
                        <p class="font-bold">${likes}<span class="font-light">Me gusta</span></p>
                        <!--Los enlaces son tipo inline por eso no toma todas las class-->
                        <p class="font-bold">${views}<span class="font-light">Veces vista</span></p>
                        
                        <a
                        class="block w-full bg-blue-800 hover:bg-blue-500 text-white uppercase font-bold text-center rounded mt-5 p-1" 
                        href="${largeImageURL}" target="_blank" rel="noopener noreferrer">Ver Imagen</a>
                     </div>

                </div>
            </div>
        `
    });
    // Limpiar el paginador previo
    while(paginacionDiv.firstChild){
        paginacionDiv.removeChild(paginacionDiv.firstChild);
    }
    // Generamos el nuevo HTML
    imprimirPaginador();
    

}


function imprimirPaginador(){
    iterador = crearPaginador(totalPaginas);

    while(true){
        const {value, done} = iterador.next();
        // Si ya llegaste al final ya no generes nada
        if(done) return;
        // caso contrario, genera un boton por cada elemento en el generador;

        const boton = document.createElement('a');
        boton.href = '#';
        boton.dataset.pagina = value;
        boton.textContent = value;
        boton.classList.add('siguiente', 'bg-yellow-400', 'px-4','py-1','mr-2', 'font-bold', 'mb-5', 'uppercase', 'rounded','justify-center');

        boton.onclick = () => {
            paginaActual = value;

            buscarImagenes();
        }


        paginacionDiv.appendChild(boton)

    }
}
