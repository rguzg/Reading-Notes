# Guía de estudio del segundo parcial

## Sitios Seguros
- ¿Qué es HTTPS?
    - HTTPS es una versión encriptada del protocolo HTTP
    - HTTPS se apoya de los protocolos SSL y TLS para encriptar la comunicación entre el cliente y el servidor
- ¿Qué es TLS?
    - TLS es el protocolo que utiliza HTTPS en la actualidad para encriptar la conexión cliente-servidor
    - TLS utiliza llaves públicas y privadas para encriptar la conexión
- ¿Qué es SSL?
    - SSL es el antecesor de TLS. Ambos fueron desarrollados por Netscape en los 90s
    - No hay mucha diferencia entre SSL y TLS. TLS solamente fue creado para separar el nombre de Netscape
- ¿Cómo funciona el TLS Handshake?
    1. El cliente y el servidor se ponen de acuerdo sobre que versión TLS van a utilizar
    1. El servidor comparte su llave pública utilizando el certificado del servidor
    1. El cliente verifica la identidad del servidor utilizando el certificado
    1. El cliente y el servidor deciden que tipo de cifrado van a utilizar y utilizan la llave pública para crear una llave de sesión que servirá para encriptar los contenidos de la sesión
## Remote Procedure Calls
- ¿Qué son las remote procedure calls?
    - Las remote procedure calls permiten ejecutar código de forma remota
    - Un cliente le solicita a un servidor que solicite una función en específico recibiendo los resultados de esa ejecución
- ¿Cuál es el rol de los protocolos en las RPCs?
    - Los protocolos son utilizados para definir una manera estándar en la que se va a recibir la petición y en la que se va a enviar la respuesta
    - Utilizar protocolos permite ignorar elementos de la arquitectura de la otra computadora y concentrarse en enviar/recibir llamadas
- ¿Es HTTP un protocolo de RPC?
    - Si, ya que HTTP permite enviarle a otra computadora una petición para que realize una acción en especifico. Además de eso, HTTP define una manera en que las dos computadoras realizaran las peticiones y enviarán los resultados de la operación
## DCOM, DCL, EJB
- ¿Qué es COM?
    - Computer Object Model es una tecnología desarrollada por Microsoft para que objetos de diferentes procesos se comuniquen entre sí
    - COM utiliza interfaces para realizar la comunicación entre procesos. Una aplicación define una interfaz con un conjunto de funciones que otros procesos pueden utilizar; estas funciones llaman a funciones internas del proceso que procesan la petición y le retornan los resultados al otro cliente
    - Un ejemplo de COM es la capacidad de manipular archivos de Excel dentro de Word. Excel expone una interfaz para manipular archivos y Word la utiliza para implementar esta función
- ¿Qué es DCOM?
    - Distributed COM es una extensión del protocolo COM que permite interactuar con objetos que se encuentren en otras computadoras dentro de la misma red
    - El cliente DCOM utiliza RPCs para comunicarse con el servidor COM de la otra computadora
- ¿Qué es DCE?
    - Distributed Computing Environment es un framework para la creación de aplicaciones cliente-servidor
    - Uno de los objetivos de DCE era estandarizar la manera en la que un cliente y un servidor se comunicaban entre sí
- ¿Qué componentes tiene DCE?
    - Componente de RPC
    - Componente de hilos: Permite desarrollar aplicaciones concurrentes que utilcen la arquitectura cliente-servidor. Contiene objetos de sincronización y un thread scheduler
    - Componente de seguridad: Servicio de autenticación, autorización y permisos para acceder a recursos
    - Distributed File System: Permite que multiples personas puedan acceder a un archivo al mismo tiempo. Las peticiones de acceso sutilizan hilos y el sistema de directorios utiliza RPCs
    - Componente de tiempo: Sincroniza el tiempo entre todos los clientes
- ¿Qué es EJB?
    - Enterprise Java Beans son componentes de servidor que permiten desarrollar operaciones comunes para aplicaciones web de manera rápida
    - Los EJB tienen la capacidad de comunicarse con otras objetos de otras aplicaciones Java. Esto puede ser utilizado, por ejemplo, para separar la lógica de negocio de la lógica que implementa el servidor para comunicarse con el cliente
- ¿Qué tipos de EJB existen?
    - Stateful Session Beans: Mantienen la información de sesión de un cliente
    - Stateless Session Beans: Estos no mantienen iformación de sesión y son utilizados para almacenar conexiones a BD o referencias a otros EJB
    - Singleton Session Beans: Solo puede existir uno de estos beans por aplicación. Almacenen información que puede ser utilizada por otros beans
    - Message-Driven Beans: Beans que envián respuestas asíncronas a clientes
## Arquitecturas de Procesamiento en Cluster
- ¿Qué son los clusters?
    - Una colección de computadoras interconectadas trabajando de manera cooperativa sobre un mismo recurso
    - Los clusters son utilizados en aplicaciones cientificas, de comercio y como servidores web o de bases de datos
- ¿Qué es el modelo de memoria distribuido?
    - En este modelo de memoria, multiples procesadores pueden compartir datos entre si, pero no pueden acceder directamente a la memoria de otros procesadores. Los procesadores trabajan de manera independiente y es tarea del desarrollador definir como y cuando se compartirán datos los procesadores
- ¿Qué es MPI?
    - Message Passing Interface es un protocolo que define la manera en que los procesadores de un cluster se comunican entre sí
    - MPI define dos tipos de clusters: cluster maestro y los trabajadores
    - MPI también contiene herramientas para dividir tareas entre trabajadores
- ¿Cómo es la arquitectura de los programas que utilizan MPI?
    - La arquitectura de los programas que utilizan MPI está dividia en dos partes:
        - Parte Serial: Contiene código que no se ejecutará de manera paralela.
        - Parte Paralela: Contiene código que se ejecutara de manera paralela. Esta sección está normalmente delimitada por las funciones MPI_Init y MPI_Finalize
## Gestión y Taxonomía de Procesos
- ¿Qué es un proceso?
    - Un contenedor para un conjunto de recursos que son utilizados cuando se está ejecutando la instancia de un programa
- ¿Qué partes componen a un proceso?
    - Un espacio de memoria virtual privado que contiene código y datos utilizados por el programa
    - Una lista de recursos del sistema que está utilizando el procesos
    - Un apuntador al proceso padre del proceso
    - Una lista de variables de entorno
    - Al menos un hilo de ejecución
- ¿Qué es un hilo?
    - Es la unidad ejecutable básica de un programa
    - Los hilos pueden ejecutar código, crear nuevos procesos, crear nuevos y hilos y gestionar la comunicación y sincronización entre los hilos de un proceso
    - Todos los hilos comparten el mismo espacio de memoria virtual y tienen acceso a todos los recursos que posea un recurso