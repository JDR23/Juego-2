#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>

using namespace std;


#define RED     "\033[41m \033[0m" // Círculo rojo
#define GREEN   "\033[42m \033[0m" // Círculo verde
#define YELLOW  "\033[43m \033[0m" // Círculo amarillo
#define BLUE    "\033[44m \033[0m" // Círculo azul
#define BOLD    "\033[1m"          // Negrita


class Celda {
public:
    string color; 
    bool ocupada;

    Celda() : color(" "), ocupada(false) {}

    void mostrar() {
        if (ocupada)
            cout << color << "|"; 
        else
            cout << " |"; 
    }

    void colocarFicha(const string& colorJugador) {
        color = colorJugador;
        ocupada = true;
    }

    void quitarFicha() {
        color = " ";
        ocupada = false;
    }
};


class Tablero {
private:
    int guia[11] = {3, 5, 7, 9, 11, 13, 11, 9, 7, 5, 3};

public:
    Celda** ptr;

    Tablero() {
        ptr = new Celda*[11];
        for (int j = 0; j < 11; j++) {
            ptr[j] = new Celda[guia[j]];
        }
    }

    void mostrar() {
        for (int i = 0; i < 11; i++) {
            for (int j = 0; j < guia[i]; j++) {
                ptr[i][j].mostrar();
            }
            cout << endl;
        }
    }

    void colocarFicha(int fila, int columna, const string& colorJugador) {
        if (fila < 0 || fila >= 11 || columna < 0 || columna >= guia[fila]) {
            cout << "Posición inválida." << endl;
            return;
        }
        if (ptr[fila][columna].ocupada) {
            cout << "La celda está ocupada. La ficha del jugador anterior será desplazada." << endl;
            // Desplazar la ficha anterior
            for (int i = guia[fila] - 1; i >= 0; --i) {
                if (ptr[fila][i].ocupada) {
                    ptr[fila][i].quitarFicha();
                    break;
                }
            }
        }
        ptr[fila][columna].colocarFicha(colorJugador);
    }
};


class Ficha {
public:
    string color;
    Ficha(const string& color) : color(color) {}
};


class Jugador {
public:
    string nombre;
    Ficha ficha;

    Jugador(const string& nombre, const string& color) : nombre(nombre), ficha(color) {}

    void colocarFicha(Tablero& tablero, int fila, int columna) {
        tablero.colocarFicha(fila, columna, ficha.color);
    }
};

// Función para simular el lanzamiento de dados (retorna un número entre 1 y 6)
int lanzarDados() {
    return rand() % 6 + 1;
}


void iniciarJuego() {
    srand(time(0)); 

    
    cout << BOLD << "\n\t\t\tBienvenido a: Can't Stop\n" << "\033[0m" << endl;

    
    Tablero tablero;
    tablero.mostrar(); // Tablero inicial vacío

  
    int numJugadores;
    do {
        cout << "Ingrese el número de jugadores (1-4): ";
        cin >> numJugadores;
        if (numJugadores < 1 || numJugadores > 4) {
            cout << "Error: El número de jugadores debe estar entre 1 y 4." << endl;
        }
    } while (numJugadores < 1 || numJugadores > 4);

    
    vector<string> colores = {RED, BLUE, GREEN, YELLOW};
    vector<Jugador> jugadores;


    for (int i = 0; i < numJugadores; i++) {
        string nombre = "Jugador " + to_string(i + 1);
        cout << "El " << nombre << " ha elegido el color: " << colores[i] << endl;
        jugadores.push_back(Jugador(nombre, colores[i]));
    }

    // Ciclo de turnos
    int turno = 0;
    while (true) {
        Jugador& jugadorActual = jugadores[turno % numJugadores];

        // Mostrar tablero antes del turno del jugador
        cout << "Turno de " << jugadorActual.nombre << endl;
        tablero.mostrar();


        int dado1 = lanzarDados();
        int dado2 = lanzarDados();
        int dado3 = lanzarDados();
        int dado4 = lanzarDados();

        cout << jugadorActual.nombre << " lanzó " << dado1 << ", " << dado2 << ", " << dado3 << " y " << dado4 << endl;

        // Selección de dados a sumar por el jugador
        int eleccion1, eleccion2;
        bool seleccionValida = false;
        bool sumaValida = false;
        while (!seleccionValida) {
            cout << "Seleccione los valores de los dados que desea sumar (1, 2, 3 o 4): " << endl;
            cout << "Primera elección: ";
            cin >> eleccion1;
            cout << "Segunda elección: ";
            cin >> eleccion2;


            if ((eleccion1 >= 1 && eleccion1 <= 4) &&
                (eleccion2 >= 1 && eleccion2 <= 4) &&
                (eleccion1 != eleccion2)) {
                seleccionValida = true;

                
                int sumaSeleccionada = (eleccion1 == 1 ? dado1 : eleccion1 == 2 ? dado2 : eleccion1 == 3 ? dado3 : dado4) +
                                       (eleccion2 == 1 ? dado1 : eleccion2 == 2 ? dado2 : eleccion2 == 3 ? dado3 : dado4);
                
                if (sumaSeleccionada <= 10) { // Suponiendo que las sumas válidas están en un rango de 2 a 12
                    sumaValida = true;
                    cout << "La suma seleccionada es: " << sumaSeleccionada << endl;
                    
                    // Seleccionar posición para colocar ficha en base a la suma
                    int fila, columna;
                    cout << "Con la suma seleccionada (" << sumaSeleccionada << "), elija fila (0-10): ";
                    cin >> fila;
                    cout << "Elija columna para la fila seleccionada: ";
                    cin >> columna;

                    // Colocar la ficha en el tablero
                    jugadorActual.colocarFicha(tablero, fila, columna);
                    // Mostrar el tablero después de colocar la ficha
                    tablero.mostrar();
                }
            } else {
                cout << "Selección inválida. Asegúrese de elegir dos dados diferentes entre 1 y 4." << endl;
            }
        }

        // Si no fue una suma válida, el jugador pierde su avance
        if (!sumaValida) {
            cout << "No se pudo utilizar la suma. Has perdido tu avance en este turno." << endl;
        }

        // Cambiar al siguiente jugador
        turno++;
    }
}

int main() {
    iniciarJuego();
    return 0;
}
