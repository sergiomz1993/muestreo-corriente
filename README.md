/////# muestreo-corriente

#include "mbed.h"
#include "TextLCD.h"
#define E  p6
//#define RW p6
#define RS p5
#define D4 p7
#define D5 p8
#define D6 p9
#define D7 p10
#define sensor p16

#define muestra  180
DigitalOut led(LED1);
//TextLCD lcd(p15, p16, p17, p18, p19, p20); // rs, e, d4-d7
//TextLCD lcd(RS, E, D4, D5, D6, D7); // rs, e, d4-d7
Ticker sample;
AnalogIn Sensor(sensor);
Serial pc(USBTX,USBRX);
uint16_t k=0;
uint16_t DatosSensor[muestra];


void lectura()
{
    if (k<muestra) {
        DatosSensor[k] = Sensor.read_u16(); // Leer los datos del Sensor.
        k++;
    }
}

int max(uint16_t* arr, int tam, int primero, int ultimo)
{
    uint16_t m=(primero+ultimo)/2;
    if (arr[m]>=arr[m+1] && arr[m] >= arr[m-1]) {
        return m;
    } else if (arr[m] < arr[m+1]) {
        primero=m+1;
    } else if (arr[m]<arr[m-1]) {
        ultimo=m-1;
    }
    return max(arr,tam,primero,ultimo);
}

int min(uint16_t* arr, int tam, int primero, int ultimo)
{
    uint16_t m=(primero+ultimo)/2;
    if (arr[m]<=arr[m+1] && arr[m] <= arr[m-1]) {
        return m;
    } else if (arr[m] > arr[m+1]) {
        primero=m+1;
    } else if (arr[m]>arr[m-1]) {
        ultimo=m-1;
    }
    return min(arr,tam,primero,ultimo);
}


int main()
{
    sample.attach(&lectura, 0.00037);
    while(1) {
        if(k==muestra) {
            sample.detach();
            uint16_t Ipi_max= DatosSensor[max(DatosSensor, muestra, 0,muestra-1)];
            uint16_t Ipi_min= DatosSensor[min(DatosSensor, muestra, 0,muestra-1)];
            lcd.cls();
            pc.printf("Ip: %1.2f A\n",((Ipi_max-Ipi_min)*0.000272)/2);            
            lcd.printf("Ip: %1.2f A\n",((Ipi_max-Ipi_min)*0.000272)/2);
            pc.printf("Irms: %1.2f A\n",(((Ipi_max-Ipi_min)*0.000272)/2)/1.414);
            lcd.printf("Irms: %1.2f A",(((Ipi_max-Ipi_min)*0.000272)/2)/1.414);
            k=0;
            sample.attach(&lectura, 0.00037);
        }
        led= !led;
        wait(0.5);

    }
}
