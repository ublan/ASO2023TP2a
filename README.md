# ASO2023TP2a

#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define NUMBER_OF_THREADS 2
#define CANTIDAD_INICIAL_HAMBURGUESAS 20

int cantidad_restante_hamburguesas = CANTIDAD_INICIAL_HAMBURGUESAS;
int turno = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void *comer_hamburguesa(void *tid) {
    int thread_id = *((int *)tid);
    
    while (1) {
        pthread_mutex_lock(&mutex);
        if (cantidad_restante_hamburguesas > 0 && turno == thread_id) {
            printf("Hola! soy el hilo(comensal) %d, me voy a comer una hamburguesa! Todavía quedan %d\n", thread_id, cantidad_restante_hamburguesas);
            cantidad_restante_hamburguesas--;
            turno = 1 - thread_id; // Cambia el turno al otro hilo
        } else if (cantidad_restante_hamburguesas <= 0) {
            printf("SE TERMINARON LAS HAMBURGUESAS :(\n");
            pthread_mutex_unlock(&mutex);
            pthread_exit(NULL);
        }
        pthread_mutex_unlock(&mutex);
    }
}

int main(int argc, char *argv[]) {
    pthread_t threads[NUMBER_OF_THREADS];
    int thread_ids[NUMBER_OF_THREADS] = {0, 1};
    int status, i;

    for (int i = 0; i < NUMBER_OF_THREADS; i++) {
        printf("Hola!, soy el hilo principal. Estoy creando el hilo %d\n", i);
        status = pthread_create(&threads[i], NULL, comer_hamburguesa, (void *)&thread_ids[i]);
        if (status != 0) {
            printf("Algo salió mal al crear el hilo. Código de error: %d\n", status);
            exit(-1);
        }
    }

    for (i = 0; i < NUMBER_OF_THREADS; i++) {
        void *retval;
        status = pthread_join(threads[i], &retval);
        if (status != 0) {
            printf("Error al esperar por el hilo. Código de error: %d\n", status);
            exit(-1);
        }
    }

    pthread_exit(NULL);
}

