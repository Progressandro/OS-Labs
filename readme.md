## Sistemas Operativos. Taller #1.

# Librería POSIX para el lenguaje C

Las librerías de hilos POSIX son una API estándar para la creación y manejo de hilos en C. POSIX significa Portable Operating System Interface (for Unix).  Es un estándar orientado a facilitar la creación de aplicaciones aplicaciones confiables y portables. La mayoría de las versiones populares de UNIX ( Linux, Mac OS X) cumplen este estándar en gran medida. La biblioteca para el manejo de hilos en POSIX es pthread.


# Manejo de Threads
Un paquete de manejo de hilos generalmente incluye funciones para: crear y destruir un hilo, itineración, forzar exclusión mutua y espera condicionada.
Los hilos de un proceso comparten variables globales, descriptores de archivos abiertos, y puede cooperar o interferir con otros hilos.
Todas las funciones de hilos del POSIX  comienzan con pthread. Entre ellas están:

| Función POSIX  	| Descripción (páginas man, otra)                          	|
|----------------	|----------------------------------------------------------	|
| pthread_equal  	| verifica igualdad de dos identificados de hilos          	|
| pthread_self   	| retorna ID de propio hilo (análogo a getpid)             	|
| pthread_create 	| crea un hilo (análogo a fork)                            	|
| pthread_exit   	| termina el hilo sin terminar el proceso (análogo a exit) 	|
| pthread_join   	| espera por el término de un hilo (análogo a waitpid)     	|
| pthread_cancel 	| Termina otro hilo  (análogo a abort)                     	|
|                	|                                                          	|
| pthread_detach 	| Configura liberación de recursos cuando termina          	|
| pthread_kill   	| envía una señal a un hilo                                	|


## Modo de uso.
> Para hacer uso de estas funciones se debe incluir la librería `<pthread.h>`.

## Compilación.

```c
$(COMPILADOR) [Windows: -lpthread | Unix: -pthread] example.c -o example.o
./example.o
```

## Identificación de Hilos

> Así como un proceso tiene un PID (Process Identification), cada hilo tiene un identificador de hilo. Mientras los PID son enteros no negativos,  el ID de un hilo es  dependiente del SO y puede ser una estructura. Por esto para su comparación se usa una función.

```
#include <pthread.h>

// Retorna 1 si son iguales, 0 en otro caso.

int pthread_equal(pthread_t tid1, pthread_t tid2);



// Retorna el identificador del hilo que invocó la función.

pthread_t pthread_self(void);
```

## Creación de Hilos
> Los procesos normalmente corren como un hilo único. La creación de un nuevo hilo se logra vía pthread_create.

```
# include <pthread.h>

// tidp: salida, puntero a id del hilo
// attr: entrada, para definir atributos del hilo, null para default
// start_routine: entrada, función a correr por el hilo
// arg: entrada, argumento de la función del hilo.
// La función debe retornar un * void, el cual es interpretado
// como el estatus de término por pthread_join

int pthread_create(pthread_t * restrict tidp,
const pthread_attr_t * restrict attr,
void * ( * start_routine ) (void *),
void * restrict arg);
```

## Término de un Hilo

> Si un hilo invoca a exit, _Exit o _exit, todo el proceso terminará.  
>
> Un hilo puede terminar de tres maneras sin terminar el proceso: Retornando de su rutina de inicio, cancelado por otro hilo del mismo proceso, o llamando pthread_exit.

```
#include <pthread.h>

// rval_ptr queda disponible para otros hilos al llamar pthread_join
// rval_ptr debe existir después del término del hilo.

void pthread_exit (void * rval_ptr);



// El hilo llamante se bloquea hasta el término del hilo indicado.
// Si el hilo en cuestión es cancelado, rval_prt toma el valor PTHREAD_CANCELED
// Si no estamos interesados en el valor retornado, poner NULL.

int pthread_join(pthread_t tid, void ** rval_ptr);



// Permite a un hilo cancelar otro hilo del mismo proceso.
// Retorno 0 es OK, !=0 => error.
// Equivale a si el hilo indicado llamara
// pthread_exit(PTHREAD_CANCELED); sin embargo, un hilo puede
// ignorar este requerimiento o controlar cómo se cancela.
// pthread_cancel sólo hace un requerimiento, pero no lo fuerza.

int pthread_cancel(pthread_t tid);



// Permite cambiar el estado del hilo a
// PTHREAD_CANCEL_ENABLE (default) o
// PTHREAD_CANCEL_DISABLE, en este estado el hilo ignora
// llamados a pthread_cancel que le afecten.

int pthread_setcacelstate(int state, int * oldstate);
```

## Detaching(Separación) y Joining(Unión) de Hilos

> Todo hilo ocupa recursos del SO para su operación. Entre ellos se encuentra el estatus de término el cual es retenido hasta el llamado a pthread_join; sin embargo, los recursos ocupados por un hilo pueden ser retornados inmediatamente después que éste termina si llamamos a pthread_detach. En este caso un llamado a pthread_join fallará y retornará EINVAL.

```
#include <pthread.h>

// retorna 0 es OK, !=0 => error.
// al término del hilo con tid, sus recursos serán retornados
// y llamados a pthread_join arrojan error.

int pthread_detach(pthread_t tid);
```

# Manejo de Sincronización(Mutex)

> Estas funciones incluyen mecanismos de exclusión mutua (mutex), mecanismos de señalización del cumplimiento de condiciones por parte de variables, y mecanismos de acceso de variables que se modifican en forma exclusiva, pero pueden ser leidas en forma compartida. Las funciones para el manejo de zonas de acceso exclusivo tienen el prefijo `pthread_mutex`.
>
>Un mutex es una variable especial que puede tener estado tomado (locked) o libre (unlocked). Es como una compuerta que permite el acceso controlado. Si un hilo tiene el mutex entonces se dice que es el dueño del mutex. Si ningún lilo lo tiene se dice que está libre (o unclucked). Cada mutex tiene una cola de hilos que están esperando para tomar el mutex. El uso de mutex es eficiente, pero debería ser usado sólo cuando su acceso es solicitado por corto tiempo.

| Función POSIX         	| Descripción                                                                                                         	|
|-----------------------	|---------------------------------------------------------------------------------------------------------------------	|
| pthread_mutex_destroy 	| Destruye la variable (de tipo pthread_mutex_t) usada para manejo de explusión mutua, o candado mutes                	|
| pthread_mutex_init    	| permite dar las condiciones iniciales a un candado mutex                                                            	|
| pthread_mutex_lock    	| Permite solicitar acceso al mutex, el hilo se bloquea hasata su obtención                                           	|
| pthread_mutex_trylock 	| permite solicitar acceso al mutex,  el hilo retorna inmediatamente. El valor retrnado indica si otro hilo lo tiene. 	|
| pthread_mutex_unlock  	| Permute liberar un mutex.                                                                                           	|

## Creación e Inicialización de mutex

```
#include <pthread.h>

// Creación

int  pthread_mutex_init(pthread_mutex_t * mutex, const pthread_mutexattr_t * attr);


// Alternativamente podemos invocar:

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

// lo cual inicializa el mutex con los atributos por omisión. 
// Es equivalente a invocar:

pthread_mutex_t mylock;
pthread_mutex_init(& mylock, NULL);

```

## Destrucción de un mutex

```
#include <pthread.h>

//Destrucción

int  pthread_mutex_destroy(pthread_mutex_t * mutex);

// Si el mutex lo tenía otro hilo y éste es destruido,
// POSIX no define el comportamiento del mutex en esta situación.
```

## Solicitud y Liberación de un mutex

```
#include <pthread.h>

int  pthread_mutex_lock(pthread_mutex_t * mutex);
int  pthread_mutex_trylock(pthread_mutex_t * mutex);
int  pthread_mutex_unlock(pthread_mutex_t * mutex);


// Con pthread_mutex_trylock el hilo siempre retorna,
// si la función es exitosa, se retorna 0 -como en los otros casos;
// si no se retornará EBUSY indicando que otro hilo tiene el mutex.
// Ejemplo: Para protejer una zona crítica, usar:

pthread_mutex_t mylock = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&mylock);
// Sección crítica
pthread_mutex_unlock(&mylock);
```


# Ejemplos Adicionales

> Los ejemplos adicionales se encuentran en la carpeta `/examples`.  
>  
> <b>`mutex.c`</b>: Manejo de Sincronización.  
> <b>`pthread.c`</b>: Manejo de Threads.    
